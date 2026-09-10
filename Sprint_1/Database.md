# ScyllaDB Table Guide

## 1. Column Data Types (Kis type ka data store kar sakte ho)

### Basic Types

```sql
CREATE TABLE example1 (
  id UUID PRIMARY KEY,
  name TEXT,           -- string data
  age INT,              -- integer
  price DECIMAL,         -- precise decimal (money ke liye)
  is_active BOOLEAN,      -- true/false
  score DOUBLE,          -- floating point
  big_number BIGINT,       -- bada integer
  created_at TIMESTAMP,      -- date + time
  today DATE,           -- sirf date
  login_time TIME         -- sirf time
);
```

### Special/Advanced Types

```sql
CREATE TABLE example2 (
  id UUID PRIMARY KEY,
  profile_pic BLOB,          -- binary data (images, files)
  ip_address INET,           -- IP address store karne ke liye
  duration DURATION,          -- time interval
  counter_col COUNTER         -- sirf increment/decrement ke liye (special table)
);
```

---

## 2. Collection Types (Multiple values ek column mein)

```sql
CREATE TABLE user_profile (
  user_id UUID PRIMARY KEY,
  hobbies SET<TEXT>,              -- unique values ka set
  scores LIST<INT>,               -- ordered list, duplicates allowed
  preferences MAP<TEXT, TEXT>          -- key-value pairs
);
```

Example insert:

```sql
INSERT INTO user_profile (user_id, hobbies, scores, preferences)
VALUES (uuid(), {'cricket', 'coding'}, [90, 85, 100], {'theme': 'dark', 'lang': 'hindi'});
```

---

## 3. User-Defined Types (UDT) - Custom Structure

Apna khud ka data type bana sakte ho:

```sql
CREATE TYPE address (
  street TEXT,
  city TEXT,
  pincode TEXT
);

CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  name TEXT,
  home_address address    -- custom type use kar rahe hain
);
```

---

## 4. Table Design Patterns (Use-case ke hisaab se format)

### A) Simple Key-Value Table

```sql
CREATE TABLE sessions (
  session_id UUID PRIMARY KEY,
  user_id UUID,
  data TEXT
);
```

👉 Simple lookups ke liye — jaise session store.

### B) Time-Series Table (Wide Row Pattern)

```sql
CREATE TABLE sensor_data (
  sensor_id UUID,
  reading_time TIMESTAMP,
  temperature DOUBLE,
  PRIMARY KEY (sensor_id, reading_time)
) WITH CLUSTERING ORDER BY (reading_time DESC);
```

👉 IoT, logs, chat messages jaise time-based data ke liye best.

### C) Counter Table (Sirf counting ke liye)

```sql
CREATE TABLE page_views (
  page_id UUID PRIMARY KEY,
  views COUNTER
);
```

👉 Likes, views, votes count karne ke liye. **Rule:** Counter column wali table mein sirf counter columns hi ho sakte hain (koi normal column mix nahi kar sakte).

Update karne ka tareeka:

```sql
UPDATE page_views SET views = views + 1 WHERE page_id = <uuid>;
```

### D) Materialized View (Secondary access pattern)

Original table ke data ko alag key se query karne ke liye:

```sql
CREATE MATERIALIZED VIEW users_by_email AS
  SELECT * FROM users
  WHERE email IS NOT NULL AND user_id IS NOT NULL
  PRIMARY KEY (email, user_id);
```

👉 Jab tumhe same data ko alag column se search karna ho.

### E) Static Column Table

```sql
CREATE TABLE orders (
  customer_id UUID,
  order_id TIMEUUID,
  customer_name TEXT STATIC,   -- ye pure partition ke liye ek hi value
  item TEXT,
  PRIMARY KEY (customer_id, order_id)
);
```

👉 `STATIC` column har partition mein ek hi baar store hota hai, chahe kitne bhi rows ho us partition mein — storage save hoti hai.

---

## Quick Summary Table

| Type | Kab Use Karo |
|------|-------------|
| Basic types (TEXT, INT, etc.) | Normal fields |
| Collections (SET/LIST/MAP) | Multiple values ek field mein |
| UDT | Complex nested structure (address, etc.) |
| Wide-row/Time-series | Logs, messages, sensor data |
| Counter table | Likes, views, counting |
| Materialized View | Alag key se same data query karna |
| Static column | Partition-level shared data |

| **Command**                            | **Purpose**                       |
| -------------------------------------- | --------------------------------- |
| `scylla --version`                     | Show ScyllaDB version             |
| `scylla_setup`                         | Configure the system and ScyllaDB |
| `cqlsh`                                | Open the CQL shell                |
| `nodetool status`                      | Show cluster status               |
| `nodetool info`                        | Show node information             |
| `nodetool describecluster`             | Show cluster information          |
| `nodetool compactionstats`             | Show compaction status            |
| `nodetool tablestats`                  | Show table statistics             |
| `sudo systemctl status scylla-server`  | Check ScyllaDB service            |
| `sudo systemctl restart scylla-server` | Restart ScyllaDB service          |
| `journalctl -u scylla-server`          | View ScyllaDB service logs        |
| `ss -lntp`                             | Check listening ports             |

---
