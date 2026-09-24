Summary — Observed Architecture
#  1. Employee API (Go)
Framework: Gin
Database: ScyllaDB (employee_info table)
Cache: Redis (optional, enabled: false by default)
Config: config.yaml + migration.json
Migration tool: golang-migrate
Port: 8080
Endpoints: /api/v1/employee/* (create, search, search/all, search/location, search/designation, health, health/detail)
Swagger: /swagger/index.html
Metrics: /metrics (via gin-metrics/Prometheus)


# 2. Salary API (Java + Spring Boot)
Build tool: Maven
Database: ScyllaDB/Cassandra (employee_salary table)
Cache: Redis (@Cacheable annotation use ho raha hai)
Config: application.yml
Migration tool: golang-migrate (yahan bhi, Liquibase nahi)
Endpoints: /api/v1/salary/* (create/record, search, search/all)
Health: /actuator/health, Metrics: /actuator/prometheus
Swagger: /salary-documentation


# 3. Attendance API (Python + Flask)
Package manager: Poetry
Database: PostgreSQL (records table) — ye alag hai baaki services se
Cache: Redis
Migration tool: Liquibase (db.changelog-master.xml, liquibase.properties)
Endpoints: /api/v1/attendance/*
Swagger: /apidocs


# 4. Notification Worker (Python)
Scheduled job (har ghante/mahine ke start me chalta hai schedule library se)
Elasticsearch se employee data fetch karta hai, phir SMTP se salary slip email bhejta hai
Koi REST API nahi — sirf background worker hai

# 5. Frontend (ReactJS)
tabler-react UI framework use ho raha hai
Saari APIs (/employee/*, /attendance/*, /salary/*, /notification/send) ko fetch() se call karta hai
Charts ke liye C3Chart (donut charts — role/location/status distribution)
PDF generate karne ke liye kendo-react-pdf
Cross-cutting observation
Employee-API aur Salary-API dono ScyllaDB + Redis use karte hain
Attendance-API alag hai — PostgreSQL + Redis + Liquibase
Sabka apna-apna Makefile hai jisme build, docker-build, run-migrations jaise targets hain
