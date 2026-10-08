 # Attendance API (README / Presentation Script)

## 1. Introduction (Attendance API kya hai)
Attendance API ek backend microservice hai jo employees ke daily attendance records (Present/Absent aur dates) manage karti hai. Ye Python me Flask web framework aur Gunicorn WSGI server use karke banayi gayi hai. Ye ek lightweight aur fast service hai jo OT-Microservices architecture ka hissa hai.

### 2. Tech Stack Explanation (Point-by-Point)
Python + Flask: Flask ek micro-framework hai jo HTTP routing aur REST APIs banane ke liye bahut simple aur powerful hai. Gunicorn isko production me multiple workers ke through run karne me madad karta hai.

Primary database — PostgreSQL: Attendance ka data relational data hota hai, isliye iske liye PostgreSQL use kiya gaya hai. Ye robust, ACID-compliant aur reliable hai. (psycopg2 driver use hua hai).

Cache — Redis: Flask-Caching ke through Redis connect kiya gaya hai (@cache.cached()). Agar ek hi query dobara aati hai to function execute hone ke bajaye Redis se hi answer mil jata hai.

Swagger (Flasgger): API documentation aur testing UI ke liye /apidocs endpoint set up kiya gaya hai.

Migration — Liquibase: Yaha database schema create/update karne ke liye Liquibase ka use hua hai, jo XML format me changelogs ko maintain karta hai.

# Step-by-Step Explanation
Client request bhejta hai — jaise GET /api/v1/attendance/search?id=EMP001.

Flask Router (attendance.py) request receive karta hai aur validation karta hai (query_validator).

Cache check: Kyunki endpoint pe @cache.cached() laga hai, pehle Redis cache me data dhunda jata hai.

Cache HIT: Redis se seedha JSON wapas.

Cache MISS: Function aage execute hota hai.

Database call (postgres_conn.py): SDK PostgreSQL pe SQL query chalata hai (SELECT id, name, status, date FROM records WHERE id='...').

Response: Data ko format karke (domain model me map karke) client ko wapas bhej diya jata hai, aur simultaneously Redis cache me store ho jata hai.

# Run Migrations — What Happens
Pre-requisite: PostgreSQL me attendance_db database bana hona chahiye. Ise scripts/db_init.sh script run karke banaya ja sakta hai.

make run-migrations chalate ho

Liquibase tool liquibase.properties file se DB ka connection string padhta hai.

Wo migration/db.changelog-master.xml file ko read karta hai (jisme table ka structure defined hai).

PostgreSQL me jake schema aur constraints create kar deta hai.

Database me ek databasechangelog table update ho jati hai taaki tracking rahe.


## Architecture Decisions: Database & Migration Tools

### 1. Kyun PostgreSQL (ScyllaDB ke bajaye)?

OT-Microservices architecture me humne **"Polyglot Persistence"** pattern follow kiya hai — jiska matlab hai ki har service ke data ki zarurat ke hisaab se best database choose karna.

- **Employee & Salary API (ScyllaDB):** Inka data flat aur read-heavy hota hai. Ek baar employee add ho gaya ya salary generate ho gayi, to usme complex relations nahi hote. ScyllaDB (NoSQL) is high-volume, flat data ko low latency ke saath serve karne ke liye best hai.
- **Attendance API (PostgreSQL):** Attendance ka data highly **relational aur time-series** based hota hai. Hume isme complex aggregations, date-range filters, aur queries (e.g., *"Ek employee ne pichle 30 din me kitni chhuttiyan li?"* ya *"Group by department attendance kya hai?"*) ki zarurat padti hai. PostgreSQL ek ACID-compliant relational database hai jo SQL queries, joins, aur date/time operations ko NoSQL ke comparison me jyada efficiently handle karta hai.

### 2. Kyun Liquibase (`golang-migrate` ke bajaye)?

Microservices ka ek aur rule hai ki har team/service apni tech-stack ke hisaab se best tool choose kar sakti hai:

- **Language Ecosystem:** Employee API **Go (Golang)** me bani hai, isliye waha `golang-migrate` use karna natural aur native tha. Attendance API **Python** me bani hai. Halanki `golang-migrate` ko yaha bhi CLI ke through chalaya ja sakta tha, lekin Python ecosystem aur Relational Databases (PostgreSQL) ke complex schema changes ko handle karne ke liye **Liquibase** ek proven industry standard hai.
- **Format Flexibility:** `golang-migrate` sirf raw `.sql` files support karta hai. Liquibase XML, YAML, aur JSON support karta hai (jaise hamara `db.changelog-master.xml`). XML format database-agnostic hota hai — kal ko agar PostgreSQL se MySQL par shift hona ho, to Liquibase automatically queries convert kar dega, jabki `.sql` me queries manually rewrite karni padengi.

### 3. Liquibase vs. Golang-Migrate (Difference)

| **Feature**            | **Liquibase**                                                                                                           | **Golang-Migrate**                                                          |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **Primary Ecosystem**  | Java / Python / Enterprise Stacks                                                                                       | Go (Golang) Ecosystem                                                       |
| **Script Format**      | XML, YAML, JSON, aur SQL                                                                                                | Sirf Raw SQL (`.up.sql`, `.down.sql`)                                       |
| **Database Support**   | Relational DBs ke liye highly optimized hai (auto-translates XML to specific SQL dialects).                             | Relational aur NoSQL (ScyllaDB, MongoDB) dono ke liye badhiya hai.          |
| **Tracking Mechanism** | `DATABASECHANGELOG` table me checksums ke saath track karta hai. Agar file modify hui, to error dega (strict tracking). | `schema_migrations` table me sirf current version number track karta hai.   |
| **Rollback (Undo)**    | XML me likhe gaye changes (jaise `createTable`) ka rollback Liquibase khud samajh leta hai (auto-rollback).             | Har "up" migration ke liye ek manual "down" migration SQL likhni padti hai. |
| **Complexity & Setup** | Setup thoda heavy hai, par enterprise-level complex databases ke liye best hai.                                         | Setup bahut lightweight aur fast hai.                                       |

> **Conclusion:** Attendance API me humein PostgreSQL jaise relational database ke schemas manage karne the, jiske liye Liquibase ka XML-based structure aur strict tracking zyada reliable approach thi. Wahi ScyllaDB (NoSQL) ke raw queries ke liye `golang-migrate` perfect fit tha.



### ACID-Compliant aur Reliable Database

**ACID-compliant** ka simple matlab hai ki database transactions ko **reliable aur safe way** se handle karta hai. ACID ke 4 important rules hote hain:

- **A - Atomicity (Pura ya Kuch Nahi):** Koi bhi transaction ya toh completely save hoga, ya bilkul nahi hoga. Beech me aadha transaction nahi rahega.
- **C - Consistency (Rules ki Pabandi):** Database hamesha defined rules follow karega. Agar employee ID invalid ya duplicate hai, toh database us entry ko reject kar sakta hai.
- **I - Isolation (Bina Mix-up):** Agar 100 employees ek hi time par attendance mark kar rahe hain, toh unki transactions ek dusre ke data ko incorrectly affect nahi karengi.
- **D - Durability (Ek Baar Save, Matlab Safe):** Transaction successfully commit hone ke baad, system crash ya power failure ke baad bhi committed data ko preserve kiya jata hai.

**Reliable (Bharosemand):** PostgreSQL ACID transactions support karta hai, isliye attendance jaise important data ko consistent aur reliable way me manage kiya ja sakta hai.

**Simple Example:** Agar employee ki attendance save karte waqt server crash ho jaye, toh database transaction ko incomplete state me chhodne ke bajaye rollback kar sakta hai. Aur agar transaction successfully commit ho gayi hai, toh committed data preserve rahega.

