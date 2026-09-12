
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


# Database Selection — OT-Microservices

This project follows a **polyglot persistence** approach — each microservice uses the database best suited to its data pattern, instead of forcing a single database across the entire stack.

| Point | Attendance | Salary | Employee |
|---|---|---|---|
| **Query type** | Complex (joins, aggregate, filters) | Simple lookup (id + date) | Simple lookup (id) |
| **Data relationship** | Multiple tables joined together | Standalone record | Standalone record |
| **Reporting need** | High (monthly reports, analytics) | Low | Low |
| **Write pattern** | Very frequent (daily check-in/out) | Less frequent (monthly) | Very rare (on join/update) |
| **Consistency need** | High (ACID required) | Medium | Medium |
| **Best DB** | PostgreSQL (SQL) | ScyllaDB (NoSQL) | ScyllaDB (NoSQL) |

## Why this matters

- **Attendance-API** deals with event-based, relational data that needs aggregation and cross-table analytics → best served by a **relational (SQL)** database like **PostgreSQL**.
- **Salary-API** and **Employee-API** deal with simple, standalone records accessed primarily by ID → best served by a **NoSQL** database like **ScyllaDB** for speed and horizontal scalability.
- **Redis** is used across all three services as a caching layer to reduce database load and speed up frequently repeated read requests, regardless of the underlying database technology.

# Why Liquibase (instead of manual SQL) — Attendance-API

PostgreSQL is fully capable of creating tables on its own. Liquibase isn't used because PostgreSQL "needs" it — it's used to manage, track, and automate schema changes safely across environments and teams.

## Comparison Table

| Without Liquibase (Manual SQL) | With Liquibase |
|---|---|
| Har server pe manually SQL likhna padta hai | Ek changelog file, sab jagah automated run |
| Kya already apply hua, yaad rakhna padta hai | Khud track karta hai (`DATABASECHANGELOG` table) |
| Rollback khud likhna padta hai | Built-in rollback support |
| Team ko pata nahi kya change hua | Git me versioned, sabko visible |
| Human error ka risk | Consistent, repeatable |
| Manual deployment | CI/CD me automate ho sakta hai |

## Summary

Liquibase applies the schema defined in [`db.changelog-master.xml`](./migration/db.changelog-master.xml) to PostgreSQL using the connection details in [`liquibase.properties`](./liquibase.properties). Running:

```shell
make run-migrations
```

ensures the same schema is created consistently across every environment (local, staging, production) without manual intervention.

# What is ReactJS — Frontend-Web

ReactJS is a **JavaScript library**, created by **Facebook (Meta)**, used for building the **UI (User Interface)** of web pages.

## Simple Definition

React helps build interactive, dynamic websites — where the page updates data **without a full reload** (like Gmail, Facebook, Instagram — click something and the content changes instantly, without refreshing the whole page).

## How It Works

**1. Components** — the page is broken down into small, reusable parts (e.g. Header, Button, Card, Form):

```jsx
function Button() {
  return <button>Click Me</button>;
}
```

**2. UI updates automatically on data change** — when data (state) changes, React updates only that specific part, not the entire page.

## In the OT-Microservices Context

The **Frontend-Web** app is built using React, which:
- Fetches data from **Employee-API**, **Attendance-API**, and **Salary-API**
- Displays that data to the user as **cards, tables, and forms**
- Sends data back to those APIs when the user submits a form (e.g. adding a new employee)

## Simple Analogy

- **HTML** = the structure of a house (walls, roof)
- **CSS** = the design/paint of the house (color, decoration)
- **React (JavaScript)** = the smart system inside the house (like smart lighting) — it reacts instantly to user actions, without rebuilding the whole house

## Quick Facts

| Point | Detail |
|---|---|
| Created by | Facebook (Meta) |
| Language | JavaScript |
| Purpose | Building interactive web UI |
| Why popular | Fast, reusable components, large community support |

**Bottom line:** React is a tool for building fast, interactive websites where only the changed part of the UI updates — not the whole page — which is why apps built with it feel smooth and responsive.
