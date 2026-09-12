
## Redis ke Strong Points (kyun use hota hai)
# 1. Speed — In-Memory Storage
Redis data ko RAM me store karta hai, disk pe nahi
ScyllaDB (disk-based, though very fast) se bhi Redis 10-100x faster hota hai read ke liye
Response time: microseconds me, jabki DB query me milliseconds lagte hain

## 2. Reduce Database Load
Agar har request seedha ScyllaDB pe jaye, to DB pe load badhta hai
Redis "frequently requested" data ko cache kar leta hai, isliye ScyllaDB ko baar-baar hit nahi karna padta

## 3. Low Latency for Read-Heavy Operations
Salary API me search aur search/all jaise endpoints read-heavy hain (log baar baar salary check karte hain, kam baar new record create karte hain)
Isi liye read requests ke liye Redis perfect fit hai

## 4. Simple Key-Value Structure
Redis simple hai — key daalo, value nikalo. Complex query ki zarurat nahi
Cached response (JSON) ko as-is store kar sakte hain

## 5. TTL (Time To Live) Support
Redis me data ko automatically expire karwa sakte ho (jaise 5 min baad cache clear ho jaye)
Isse stale (purana) data zyada der tak serve nahi hota


ScyllaDB ke saath Redis kyun use kiya gaya (combination ka reason)
Cheez	ScyllaDB (akela)	ScyllaDB + Redis
Har search request	Direct DB hit	Pehle cache check, fast
High traffic (bahut users)	DB pe pressure badhta hai	Redis load absorb karta hai
Response time	Milliseconds	Microseconds (cache hit pe)
Cost/Resource	Zyada CPU/disk usage DB pe	Kam DB usage, better scaling

## Simple logic:
ScyllaDB → "Source of truth" hai, permanent data yahan safe rehta hai (durability)
Redis → "Speed layer" hai, temporary fast-access copy provide karta hai (performance)

# Redis me data store kyun NAHI kar sakte (permanent tarike se)

## 1. Data Loss ka Risk (No Durability)
Redis RAM (memory) me data rakhta hai
Agar server crash ho jaye, restart ho, ya power chali jaye → sara data gayab ho sakta hai
ScyllaDB disk pe data store karta hai — server crash ho bhi jaye, data safe rehta hai

## 2. RAM Limited Hoti Hai, Disk Nahi
RAM mehenga hota hai aur limited hota hai (jaise 8GB, 16GB)
Agar lakho employees ka salary data (saalon ka history) Redis me rakhoge, RAM full ho jayegi
ScyllaDB disk pe chalta hai jo TB (terabytes) tak scale ho sakta hai, sasta bhi hai

## 3. Cost Bahut Zyada Hoga
1GB RAM ki cost, 1GB disk storage se kai guna zyada hoti hai
Sara salary data (permanent records) Redis me rakhna financially impractical hai

## 4. Complex Queries Nahi Kar Sakte
Redis simple key-value store hai (jaise: salary:123 → {amount: 50000})
Agar tumhe query karni ho jaise "sabhi employees jinki salary 50000 se zyada hai" → Redis me ye mushkil/impossible hai
ScyllaDB (SQL-like CQL) me complex filtering, sorting, indexing possible hai

## 5. Redis ka Design Hi Alag Hai
Redis banaya gaya hai temporary, fast-access cache ke liye — permanent database ke liye nahi
Agar Redis crash hota hai ya restart hota hai (bina proper persistence config ke), data wipe ho sakta hai
Real-world analogy: ScyllaDB ek almirah hai jahan sab kuch permanently rakha hai, Redis ek table drawer hai jahan jo cheez baar-baar use ho rahi hai wo hath ke pass rakh di jati hai — taaki har baar almirah tak jana na pade.

Isi wajah se ye combination microservices architecture me bahut common pattern hai: "Database for storage, Cache for speed."

# scylladb 
CREATE TABLE IF NOT EXISTS employee_salary (
    id text,
    process_date text,
    name text,
    salary float,
    status text,
    PRIMARY KEY (id, process_date)
) WITH CLUSTERING ORDER BY (process_date DESC);
