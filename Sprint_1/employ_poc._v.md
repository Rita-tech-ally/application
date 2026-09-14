# Employee API | Local Setup Without Docker

---
## Author Table

| Author | Created on | Version | Last updated by | Last edited on | PRE Reviewer | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| ------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ----------- |
| Vineet | 12-09-2026 | v1.0 | Vineet | 12-09-2026 | Team | Aryan Mishra |        |                                          |


## Table of Contents

1. [Introduction](#introduction)
2. [Objective](#objective)
3. [Architecture](#architecture)
4. [Prerequisites](#prerequisites)
5. [Clone Repository](#clone-repository)
6. [ScyllaDB Setup](#scylladb-setup)
7. [Employee API Configuration](#employee-api-configuration)
8. [Database Migration](#database-migration)
9. [Build Employee API](#build-employee-api)
10. [Run Employee API](#run-employee-api)
11. [API Verification](#api-verification)
12. [Swagger Documentation](#swagger-documentation)
13. [Verification Checklist](#verification-checklist)
14. [Troubleshooting](#troubleshooting)
15. [Best Practices](#best-practices)
16. [Conclusion](#conclusion)
17. [Contact Information](#contact-information)
18. [References](#references)


---

# 1. Introduction

Employee API is a backend microservice used for managing employee-related information.

This document explains how to set up and run the Employee API locally on Ubuntu/WSL **without using Docker**.

The setup includes:

- Go application setup
- ScyllaDB configuration
- Database migration
- Application build
- Running the Employee API
- Health check verification
- Employee data search
- Prometheus metrics verification
- Swagger API documentation verification

---

# 2. Objective

The objective of this setup is to:

- Run the Employee API locally without Docker.
- Connect the application with a locally running ScyllaDB instance.
- Apply the required database migrations.
- Build and start the Employee API.
- Verify API health and database connectivity.
- Verify employee search functionality.
- Verify Prometheus metrics.
- Verify Swagger API documentation.

---

# 3. Architecture

The local setup follows the architecture below:

<img width="459" height="553" alt="image" src="https://github.com/user-attachments/assets/cfb60fe5-88c1-4d5f-b1a2-7fb5c6371744" />

```
# 4. Prerequisites

Before starting the Employee API setup, make sure the required tools and services are installed and available on the system.

| Tool | Purpose |
|------|---------|
| Git | Clone and manage the repository |
| Go | Build and run the Employee API |
| Make | Run project commands |
| ScyllaDB | Primary database |
| cqlsh | Verify ScyllaDB connectivity |
| curl | Verify API endpoints |

## Verify Go

```bash
go version

---
```
# 5. Clone Repository

Navigate to the OT-MICROSERVICES directory:

```bash
cd /mnt/c/OT-MICROSERVICES

git clone https://github.com/OT-MICROSERVICES/employee-api.git
---
```
# 6. ScyllaDB Setup

Employee API uses ScyllaDB as the database for storing employee information.

Make sure ScyllaDB is installed and running on the local Ubuntu/WSL environment.

### Check ScyllaDB Status

Run:
<img width="1340" height="448" alt="Screenshot (504)" src="https://github.com/user-attachments/assets/b0a5c837-486e-4767-a439-1f2184fe6ed3" />

```bash
sudo systemctl status scylla-server
---
```
# 7. Employee API Configuration

The original project configuration contains Docker network IPs. For local non-Docker execution, the ScyllaDB host is changed to `localhost`.

Open the configuration file:
<img width="958" height="154" alt="Screenshot (505)" src="https://github.com/user-attachments/assets/516ffbf3-035c-4e59-9cf3-4f33ffa1294c" />

```bash
cat config.yaml
---
```
# 8. Database Migration

The migration configuration also needs to point to the local ScyllaDB instance.

Check `migration.json`:
<img width="1062" height="159" alt="Screenshot (516)" src="https://github.com/user-attachments/assets/37b28ebe-c6c4-4bb1-aa22-6cf0bf217fe6" />

```bash
cat migration.json
---
```
# 9. Build Employee API

Build the Employee API using Make:
<img width="1153" height="226" alt="Screenshot (507)" src="https://github.com/user-attachments/assets/ba389fb0-f5ff-483c-926b-37540e562859" />

```bash
make build

---
```
# 10. Run Employee API

Start the Employee API:

<img width="1305" height="147" alt="Screenshot (509)" src="https://github.com/user-attachments/assets/c287a494-5c11-4a0b-a1a6-1a9dd81d7812" />

```bash
export GIN_MODE=release
./employee-api
---
```
# 11. API Verification

Open another terminal and navigate to the project directory:

```bash
cd /mnt/c/OT-MICROSERVICES/employee-api
---
```
# 12. Swagger Documentation

Swagger is used to view and interact with the Employee API documentation. The project uses `swaggo` and `gin-swagger`.

If Swagger documentation needs to be regenerated, run:

<img width="1920" height="910" alt="Screenshot (515)" src="https://github.com/user-attachments/assets/e5538599-b038-45d3-8273-d2182e6b4dfd" />

<img width="1920" height="930" alt="Screenshot (514)" src="https://github.com/user-attachments/assets/6a803b59-8adb-4dc2-8a4d-67faf7c344d6" />


```bash
make swagger
make build

---
```
# 13. Verification Checklist

| Component / Check | Status |
|-------------------|--------|
| Git repository cloned | ✅ |
| Go installed | ✅ |
| Make installed | ✅ |
| ScyllaDB running | ✅ |
| ScyllaDB connection verified | ✅ |
| Local ScyllaDB configuration | ✅ |
| Database migration | ✅ |
| Employee API build | ✅ |
| Employee API running | ✅ |
| Health API | ✅ |
| Detailed health API | ✅ |
| ScyllaDB connectivity | ✅ |
| Employee search | ✅ |
| Prometheus metrics | ✅ |
| Swagger generation | ✅ |
| Swagger UI | ✅ |
| Swagger JSON | ✅ |
| Docker used | ❌ |

---

# 14. Troubleshooting

### API Not Starting
Check whether port `8080` is already being used:
```bash
sudo ss -ltnp | grep :8080
---
```
# 15. Best Practices

To maintain code quality and ensure a smooth local development experience, follow these best practices:

* **Configuration Management:** Avoid committing sensitive database credentials to version control. Keep your local `config.yaml` secure and do not push it if it contains real passwords.
* **Dependency Management:** Regularly run `go mod tidy` to keep your `go.mod` and `go.sum` files clean and remove unused dependencies.
* **Code Formatting:** Always format your Go code using `go fmt ./...` before pushing any changes to the repository.
* **Database Maintenance:** Since ScyllaDB is running locally, keep an eye on your disk space. Periodically clean up unnecessary test data from the `employee_db` keyspace.
* **Graceful Shutdown:** When stopping the API locally (using `Ctrl+C`), wait for the application to gracefully close database connections rather than force-killing the terminal.
* **Git Ignore:** Ensure that the generated build binary (`employee-api`), `.env` files, and local IDE configurations (like `.vscode/`) are properly added to the `.gitignore` file.
---
# 16. Conclusion

The Employee API has been successfully configured and executed locally without Docker. 
It is securely connected to a local ScyllaDB instance with all database migrations, health checks, and Swagger endpoints fully verified. 
This setup provides a reliable and complete local environment for running and testing the API.
```
```
# 17 contact-information

| **Name** | **Email Address** |
| -------- | ----------------- |
| Vineet   | [vineet.shrivastava.snaatak@mygurukulam.co](mailto:vineet.shrivastava.snaatak@mygurukulam.co) |

# 17. References

| Resource Name | Link |
| ------------- | ---- |
| **Employee API Repository** | [https://github.com/OT-MICROSERVICES/employee-api](https://github.com/OT-MICROSERVICES/employee-api) |
| **Go Documentation** | [https://go.dev/doc/](https://go.dev/doc/) |
| **ScyllaDB Documentation** | [https://docs.scylladb.com/](https://docs.scylladb.com/) |
| **Swagger Documentation** | [https://swagger.io/docs/](https://swagger.io/docs/) |
| **Swaggo** | [https://github.com/swaggo/swag](https://github.com/swaggo/swag) |
| **Gin Swagger** | [https://github.com/swaggo/gin-swagger](https://github.com/swaggo/gin-swagger) |

---
