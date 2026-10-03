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
