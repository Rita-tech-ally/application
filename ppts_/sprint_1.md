## Presentation Script — Employee API

### 1. Introduction (Employee API kya hai)

Employee API ek backend microservice hai jo company ke employees ka data manage karta hai — create karna, search karna, aur unki information (naam, designation, department, location) retrieve karna. Ye Go language me Gin framework use karke bana hai, aur REST API ke through kaam karta hai — matlab koi bhi dusra application (mobile app, web dashboard, ya koi aur service) is API ko HTTP requests bhej ke employee data access kar sakta hai.

Ye OT-Microservices architecture ka hissa hai, isliye ye ek independent, self-contained service hai jo apna khud ka database aur caching layer rakhti hai.

### 2. Tech Stack Explanation (Point-by-Point)

**Go + Gin REST API**
Application Go language me likha gaya hai, aur Gin naam ka web framework use karta hai jo HTTP requests ko handle karta hai — routing, middleware, aur JSON responses manage karta hai. Go isliye use kiya gaya kyunki ye fast, lightweight, aur concurrent requests efficiently handle karta hai.

**Primary database — ScyllaDB**
Employee ka saara data (naam, designation, address, etc.) ScyllaDB me permanently store hota hai. ScyllaDB ek high-performance, distributed NoSQL database hai jo Cassandra-compatible hai — isse horizontal scalability aur low-latency milti hai, jo microservices ke liye ideal hai.

**Cache — Redis**
Redis ek caching layer hai jo frequently-read data (jaise search results, employee counts) ko fast serve karta hai — taaki har request pe ScyllaDB tak na jaana pade. Ye cache-aside pattern follow karta hai: pehle cache check hota hai, miss hone par ScyllaDB se data aata hai aur wapas cache me save ho jata hai. Ye optional hai, config se on/off kiya ja sakta hai.

**Swagger API documentation**
Swagger ek interactive documentation tool hai jo API ke saare endpoints (search, create, health-check) ko ek web page pe dikhata hai, jaha developers bina code likhe, seedha browser se API test kar sakte hain.

**Health & dependency checks**
Application ke paas health-check endpoints hain jo batate hain ki API khud chal rahi hai ya nahi, aur ScyllaDB/Redis jaise dependencies connected hain ya nahi — ye monitoring aur troubleshooting ke liye use hota hai.

**Migration — golang-migrate**
Database ka schema (tables ka structure) manually SQL likhne ke bajaye, ek migration tool (golang-migrate) se automatically create kiya jata hai. Ye `.sql` files me likhe structure ko database pe apply karta hai, taaki schema version-controlled aur consistent rahe.


### Step-by-Step Explanation

1. **Client request bhejta hai** — jaise `GET /api/v1/employee/search?id=EMP001`

2. **Gin Router request ko receive karta hai** aur URL ke basis par sahi handler (function) ko call karta hai (`routes.go` me defined mapping ke through)

3. **API handler (`api.go`) business logic chalata hai** — sabse pehle check karta hai ki Redis caching enabled hai ya nahi

4. **Agar Redis enabled hai**, to pehle Redis me data dhoonda jata hai:
   - **Cache HIT** (data mil gaya) → seedha wahi data response me bhej diya jata hai, ScyllaDB tak jaane ki zarurat nahi
   - **Cache MISS** (data nahi mila) → agle step pe jaate hain

5. **ScyllaDB se data fetch hota hai** — actual CQL query chalti hai (jaise `SELECT * FROM employee_info WHERE id = ?`)

6. **Fresh data Redis me wapas cache hota hai** (`writeinRedis` function se), taaki agli baar yahi request aaye to seedha Redis se mil jaye

7. **Response JSON format me client ko bhej diya jata hai**

### Kyun Ye Approach Use Ki Gayi

- **Redis ScyllaDB pe load kam karta hai** — baar-baar same data ke liye database tak nahi jaana padta
- **Response time fast hota hai** jab data cache me ho (RAM se milta hai, disk se nahi)
- **ScyllaDB hamesha source of truth rehta hai** — Redis sirf temporary, disposable cache hai; agar Redis crash ho jaye, application bina kisi data loss ke chalti rahegi, bas thodi slow ho jayegi (har request ScyllaDB tak jayegi)


### Migration Tool Capabilities

golang-migrate sirf schema create karne tak limited nahi hai — ye poora migration lifecycle manage karta hai:

- **Version tracking** — database ke andar ek internal table (`schema_migrations`) maintain karta hai, jisse pata rehta hai kaunsi migrations already apply ho chuki hain, taaki same migration dobara accidentally na chale
- **Forward migration (`up`)** — naya schema apply karta hai (table create, column add, etc.)
- **Rollback (`down`)** — agar koi change galat ho ya revert karna ho, to us migration ko undo kar sakta hai, `.down.sql` file ke through
- **Sequential ordering** — migration files ko number se naam diya jata hai (`000001_...`, `000002_...`), taaki tool ko pata rahe kaunsi migration pehle aur kaunsi baad me chalani hai

## Run Migrations — What Happens

`make run-migrations` chalane par golang-migrate tool:

1. `migration.json` se database connection string leta hai
2. `migration/` folder ki `.sql` files ko **sequence me** (`000001`, `000002`...) padhta hai
3. Database me check karta hai kaunsi migrations **already apply ho chuki hain** (version tracking se)
4. Jo migrations **pending hain**, unka schema (table create/alter) database pe **apply** kar deta hai
5. Version table update kar deta hai, taaki agli baar same migration dobara na chale

Result: `employee_info` table ScyllaDB me ban jati hai, application ke use ke liye ready.

## Run Migrations — What Happens

### Pre-requisite — Keyspace Creation (Manual Step)

Migration chalane se **pehle**, ek keyspace (`employee_db`) manually create karna padta hai, kyunki migration `.sql` files sirf `CREATE TABLE` contain karti hain, `CREATE KEYSPACE` nahi:

```sql
CREATE KEYSPACE employee_db WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'replication_factor': 1
};
```

Ye step is liye manual rakha gaya hai kyunki replication settings **environment ke hisab se alag** hoti hain (dev me `1`, production me `3` ya zyada) — isliye isse code/migration se decouple rakha gaya, taaki har environment apni zarurat ke hisab se decide kare.

### `make run-migrations` Chalane Par Kya Hota Hai

1. `migration.json` se database connection string leta hai (jisme `employee_db` keyspace ka naam already hai — jo upar manually bana chuke hain)
2. `migration/` folder ki `.sql` files ko **sequence me** (`000001`, `000002`...) padhta hai
3. Database me check karta hai kaunsi migrations **already apply ho chuki hain** (version tracking se)
4. Jo migrations **pending hain**, unka schema (table create/alter) database pe **apply** kar deta hai
5. Version table update kar deta hai, taaki agli baar same migration dobara na chale

## Result: `employee_info` table ScyllaDB me ban jati hai, application ke use ke liye ready.

1. make run-migrations chalate ho
        ↓
2. migrate tool "migration.json" se ADDRESS (connection string) leta hai
        ↓
3. "migration/" folder se TABLE KA STRUCTURE (.sql files) leta hai
        ↓
4. Dono ko combine karke ScyllaDB me jata hai
        ↓
5. SCHEMA CREATE ho jata hai (table ban jati hai)

## Build Process

1. **`go mod tidy`** — dependencies ko sync karta hai (missing packages download, unused remove)

   Example:
   - Pehle: code me `go-redis` import hai, lekin `go.mod` me nahi; `gorilla/mux` listed hai, lekin use nahi ho raha
   - Baad me: `go-redis` add ho gaya, `gorilla/mux` remove ho gaya

2. **`make build`** — do steps execute karta hai:
   - `go fmt ./...` — code ko standard Go format me arrange karta hai
   - `go build -o employee-api` — saara Go source code compile karke ek single executable binary (`employee-api`) banata hai

   **Kaun si files compile hoti hain:**

   | Folder / File | Files | Kaam |
   |---|---|---|
   | `main.go` | `main.go` | Entry point, router aur server setup |
   | `api/` | `api.go`, `health.go` | Request handlers (search, create, health) |
   | `client/` | `redis.go`, `scylladb.go` | Redis aur ScyllaDB connection |
   | `config/` | `viper.go` | `config.yaml` read karna |
   | `middleware/` | `logging.go` | Request logging |
   | `model/` | `config.go`, `employee.go`, `health.go` | Data structs |
   | `routes/` | `routes.go` | Endpoints register karna |
   | `docs/` | `docs.go` | Swagger documentation (auto-generated) |

   Note: `*_test.go` files build me compile nahi hoti, ye sirf `go test` ke time chalti hain.

   **`go.mod` / `go.sum` me jo dependencies use ho rahi hain:**

   | Dependency | Kaam |
   |---|---|
   | `gin-gonic/gin` | REST API framework (routing, middleware) |
   | `scylladb/gocql` (`gocql` ko replace karta hai) | ScyllaDB se connect hone ka driver |
   | `redis/go-redis/v9` | Redis client (caching) |
   | `spf13/viper` | `config.yaml` parse karna |
   | `sirupsen/logrus` | JSON logging |
   | `penglongli/gin-metrics` | `/metrics` endpoint (Prometheus) |
   | `swaggo/swag`, `swaggo/gin-swagger`, `swaggo/files` | Swagger documentation aur UI |
   | `stretchr/testify` | Sirf testing ke liye (binary me shamil nahi hota) |

   - **`go.mod`** — in direct dependencies aur unke versions ki list
   - **`go.sum`** — in sab dependencies (aur unki indirect dependencies) ka checksum, taaki downloaded code verify ho sake

Build hone ke baad, binary ko directly run kiya ja sakta hai:
```bash
./employee-api
```


## Middleware and Routes

### Middleware

Middleware wo code hai jo **har request pe handler se pehle/baad automatically chalta hai**. `main.go` me ye register hote hain:

| Middleware | Kaam |
|---|---|
| `gin-metrics` | `/metrics` endpoint banata hai (Prometheus monitoring ke liye) |
| `gin.Recovery()` | Agar koi handler crash (panic) ho jaye, to poori application band nahi hoti, error response milta hai |
| `LoggingMiddleware()` | Har request ka record rakhta hai (method, URL, status code, latency, client IP) JSON format me |

### Routes

Routes batate hain **kaunsa URL kaunse handler function ko call karega**. `routes.go` me:

- Base path `/api/v1` ke andar `/employee` group banta hai
- Har endpoint ek handler (`api.go`, `health.go`) se map hota hai

| Route | Handler |
|---|---|
| `GET /api/v1/employee/health` | `HealthCheckAPI` |
| `GET /api/v1/employee/health/detail` | `DetailedHealthCheckAPI` |
| `POST /api/v1/employee/create` | `CreateEmployeeData` |
| `GET /api/v1/employee/search` | `ReadEmployeeData` |
| `GET /api/v1/employee/search/all` | `ReadCompleteEmployeesData` |
| `GET /api/v1/employee/search/location` | `ReadEmployeesLocation` |
| `GET /api/v1/employee/search/designation` | `ReadEmployeesDesignation` |

### Flow

```
Request → Middleware (Recovery, Logging, Metrics) → Route match → Handler → Response
```
