
<p align="center">
  <img src="./static/employee-api-logo.svg" width="280" height="280" alt="Employee API Logo">
</p>

----

# Employee API | POC Setup Without Docker

---

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 09/08/2026 | 1.1 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |

---

# Table of Contents


1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Clone Repository](#3-clone-repository)
4. [Installation and Setup](#4-installation-and-setup)
5. [Conclusion](#5-conclusion)
6. [Contact Information](#6-contact-information)
7. [References](#7-references)

---

# 1. Introduction

Employee API is a backend microservice used for managing employee-related information.
This document contains the rough steps followed during the Employee API POC setup on an AWS cloud server without using Docker.

---


# 2. Prerequisites

Before starting the Employee API setup, an AWS EC2 instance was created with **Ubuntu 24.04**.

## 2.1 AWS EC2 Instance

The following environment was used for the POC:

| Requirement      | Configuration        |
| ---------------- | -------------------- |
| **Cloud Platform**   | AWS                  |
| **Service**          | Amazon EC2           |
| **Operating System** | Ubuntu 24.04 LTS     |
| **Architecture**     | Linux AMD64 / x86_64 |
| **Application Port** | `8080`               |
| **ScyllaDB Port**    | `9042`               |
| **Redis Port**       | `6379`               |
| **SSH Port**         | `22`                 |

> **Note:** Port `8080` must be allowed in the EC2 Security Group if the Employee API needs to be accessed from outside the instance.

---

## 2.2 Required Tools and Services

The following tools/services are required for the setup:

| Tool / Service | Purpose                                   |
| -------------- | ----------------------------------------- |
| **AWS EC2**        | Provides the cloud server for the POC     |
| **Ubuntu 24.04**   | Operating system used on the EC2 instance |
| **Git**            | Clone the Employee API repository         |
| **Go**             | Build and run the Employee API            |
| **Make**           | Run project Makefile commands             |
| **ScyllaDB**       | Primary database for employee data        |
| **cqlsh**          | Connect and interact with ScyllaDB        |
| **Redis**          | Cache used by the Employee API            |
| **curl**           | Download tools and test API endpoints     |
| **wget**           | Download the Go package                   |
| **migrate**        | Run database migrations                   |
| **Swagger**        | View and test API documentation           |

---

# 3. Clone Repository

The Employee API source code was cloned from the OT-MICROSERVICES GitHub repository.

### Clone the repository

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api.git
```
<img width="1139" height="211" alt="Screenshot from 2026-09-14 12-53-44" src="https://github.com/user-attachments/assets/55cec99e-86a5-4a64-9c6c-a83229749cb3" />

### Move into the project directory

```bash
cd employee-api
```

# 4. Installation and Setup

This section covers the installation, configuration, database migration, build, and verification of the Employee API on an Ubuntu EC2 instance.

## 4.1 ScyllaDB Setup

**Install ScyllaDB:**

```bash
curl -sSf get.scylladb.com/server | sudo bash
```
<img width="1416" height="332" alt="Screenshot from 2026-09-14 12-57-46" src="https://github.com/user-attachments/assets/3b4d2df6-2a09-47fb-a432-85839feb1557" />



**Enable developer mode:**

```bash
sudo scylla_dev_mode_setup --developer-mode 1
```
<img width="1078" height="35" alt="Screenshot from 2026-09-14 14-49-14" src="https://github.com/user-attachments/assets/456d7a00-7364-4e00-a862-f189b534c6dd" />



**Enable, start and status check ScyllaDB:**

```bash
sudo systemctl enable scylla-server
sudo systemctl start scylla-server
```
<img width="1581" height="85" alt="Screenshot from 2026-09-14 12-58-33" src="https://github.com/user-attachments/assets/ff88c773-3717-4234-bf3d-44a83a1aa5b1" />



```bash
sudo systemctl status scylla-server
```
<img width="1085" height="383" alt="Screenshot from 2026-09-14 12-59-36" src="https://github.com/user-attachments/assets/249b37d2-5bbb-4835-acc3-7052ae7398c6" />




**Connect to ScyllaDB:**

```bash
cqlsh 127.0.0.1 9042
```

Create the Employee API keyspace:

```sql
CREATE KEYSPACE employee_db WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'replication_factor': 1
};
```
<img width="1527" height="345" alt="Screenshot from 2026-09-14 13-40-59" src="https://github.com/user-attachments/assets/8631ba43-6b85-42e2-8e1f-1187090c8b51" />

> **Purpose:** ScyllaDB is used as the primary database for storing Employee API data.

---

## 4.2 Redis Setup

**Install Redis:**

```bash
sudo apt install redis-server -y
```
<img width="1085" height="470" alt="Screenshot from 2026-09-14 13-01-22" src="https://github.com/user-attachments/assets/dd533828-97cd-42ad-af55-3544460050a0" />

**Enable, start and status check Redis:**


```bash
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

<img width="1087" height="106" alt="Screenshot from 2026-09-14 13-02-05" src="https://github.com/user-attachments/assets/ad1a980d-20c4-4c0b-9c86-3c02abcd6ec0" />

```bash
sudo systemctl status redis-server
```
<img width="1236" height="362" alt="Screenshot from 2026-09-14 13-02-57" src="https://github.com/user-attachments/assets/990c1210-d584-41d1-a6ae-4e2d5e097608" />


> **Purpose:** Redis is used as a caching layer for the Employee API.

---

## 4.3 Go Setup

**Download Go:**

```bash
wget https://dl.google.com/go/go1.20.14.linux-amd64.tar.gz
```
<img width="1239" height="297" alt="Screenshot from 2026-09-14 13-04-42" src="https://github.com/user-attachments/assets/7f1f42a9-e361-4a7f-aa00-9b8be1882cc5" />

**Remove any existing Go installation:**

```bash
sudo rm -rf /usr/local/go
```
<img width="1242" height="33" alt="Screenshot from 2026-09-14 13-05-28" src="https://github.com/user-attachments/assets/fbd2e201-89a1-4669-90f9-e87229a31d34" />


**Install Go:**

```bash
sudo tar -C /usr/local -xzf go1.20.14.linux-amd64.tar.gz
```
<img width="1242" height="36" alt="Screenshot from 2026-09-14 13-07-08" src="https://github.com/user-attachments/assets/497533fb-4689-47c5-83a1-c34c8815a409" />


**Add Go to `PATH`:**

```bash
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

<img width="1238" height="61" alt="Screenshot from 2026-09-14 13-06-01" src="https://github.com/user-attachments/assets/5dfcd2b1-cf37-4f6e-9ade-0ed873cc2543" />

**Verify the installation:**

```bash
go version
```
<img width="1073" height="70" alt="Screenshot from 2026-09-14 15-05-15" src="https://github.com/user-attachments/assets/78566c74-86f4-4b50-8c51-26c84759bd6c" />

> **Purpose:** Go is required to build and run the Employee API.

---

## 4.4 Employee API Configuration

Move to the application directory:

```bash
cd employee-api
```

**Edit the application configuration:**

```bash
Update the database address from the Docker IP(172.17.0.3) to:
127.0.0.1
```

```bash
cat config.yaml
```
<img width="1244" height="314" alt="Screenshot from 2026-09-14 13-08-17" src="https://github.com/user-attachments/assets/146ebcf1-4e8c-475b-bcb2-ecf845e07496" />



> **Purpose:** The Employee API, ScyllaDB, and Redis are running on the same EC2 instance, so `localhost` is used instead of the Docker IP.

---

## 4.5 Migration Configuration

**Edit the migration configuration:**
```bash
Update the database address from the Docker IP(172.17.0.3) to:
127.0.0.1
```
```bash
cat migration.json
```
<img width="1243" height="117" alt="Screenshot from 2026-09-14 13-09-15" src="https://github.com/user-attachments/assets/aeb4823a-93e9-4bdf-b948-7408e09f7112" />



> **Purpose:** Allows database migrations to connect to the local ScyllaDB instance.

---

## 4.6 Swagger Configuration

**Edit `main.go`:**

Configure the Swagger documentation URL:

```go
url := ginSwagger.URL("/swagger/doc.json")
```

```bash
cat main.go
```
<img width="1300" height="399" alt="Screenshot from 2026-09-14 13-38-57" src="https://github.com/user-attachments/assets/2717193a-94bf-4baa-a4f8-985f39479077" />




**Edit the Swagger documentation:**

Update the API host:

```go
Host: "43.204.108.146:8080",
```

```bash
cat docs/docs.go
```
<img width="891" height="409" alt="Screenshot from 2026-09-14 13-13-33" src="https://github.com/user-attachments/assets/83618a49-c1c8-4673-928a-50231fb5b83c" />



> **Purpose:** Configures Swagger to generate and access the API documentation using the EC2 public IP and application port.

---

## 4.7 Database Migration

**Download the `migrate` tool:**

```bash
curl -L https://github.com/golang-migrate/migrate/releases/download/v4.15.2/migrate.linux-amd64.tar.gz | tar xvz
```
<img width="1846" height="211" alt="Screenshot from 2026-09-14 13-14-18" src="https://github.com/user-attachments/assets/577ef807-43f5-4c4b-a6c6-15ceba664ee5" />

**Move it to the system path:**

```bash
sudo mv migrate /usr/local/bin/
```
<img width="1846" height="33" alt="image" src="https://github.com/user-attachments/assets/56af3a5c-b42d-4295-9d5a-cf49cde0f8d0" />

Verify the installation:

```bash
migrate -version
```
<img width="1073" height="70" alt="Screenshot from 2026-09-14 15-04-12" src="https://github.com/user-attachments/assets/a1a0a447-c259-4241-b8e4-2a2c2a2340ef" />

**Install Make:**

```bash
sudo apt install make
```
<img width="998" height="176" alt="Screenshot from 2026-09-14 13-16-14" src="https://github.com/user-attachments/assets/b6f7b69b-6fc7-4d67-8c0f-7b9304d88a84" />

**Run database migrations:**

```bash
make run-migrations
```
<img width="1501" height="83" alt="make" src="https://github.com/user-attachments/assets/89ec9de1-acec-4dc6-b9f4-b0edf9100545" />

> **Purpose:** Applies the required database schema and migrations to ScyllaDB.

---

### 4.8 Build Employee API

**Update Go dependencies:**

```bash
go mod tidy
```
<img width="1495" height="377" alt="Screenshot from 2026-09-14 13-17-16" src="https://github.com/user-attachments/assets/c9e9f73c-c0ea-41e6-8346-01549c5ea9f1" />

**Build the application:**

```bash
make build
```
<img width="1511" height="83" alt="Screenshot from 2026-09-14 13-17-44" src="https://github.com/user-attachments/assets/db2a4e07-44a2-43c1-b667-e5f227737527" />

> **Purpose:** Builds the Employee API executable after resolving the required Go dependencies.

---

## 4.9 Run Employee API

**Start the Employee API in the background:**

```bash
nohup ./employee-api > ~/employee.log 2>&1 &
```
<img width="1518" height="90" alt="Screenshot from 2026-09-14 13-18-14" src="https://github.com/user-attachments/assets/3945bba6-3e0b-4b57-b11d-80b5030d0da6" />

**Verify that the application is listening on port `8080`:**

```bash
ss -tulnp | grep 8080
```

> **Purpose:** Runs the API in the background and verifies that it is listening on port `8080`.

---

## 4.10 API Verification

**Verify the Employee API health endpoint:**

```bash
curl http://localhost:8080/api/v1/employee/health/detail
```
<img width="1119" height="79" alt="Screenshot from 2026-09-14 13-19-51" src="https://github.com/user-attachments/assets/4e67d015-08d8-4fd7-a175-9b77ab7e5442" />

> **Purpose:** Confirms that the Employee API is running successfully and responding to requests.

---

## 4.11 Swagger Verification


**Swagger UI:**

```text
http://43.204.108.146:8080/swagger/index.html
```


<img width="1834" height="593" alt="Screenshot from 2026-09-14 13-20-52" src="https://github.com/user-attachments/assets/b1546308-39b4-4a85-808e-2fbea37189c8" />
<img width="1870" height="975" alt="Screenshot from 2026-09-14 13-21-57" src="https://github.com/user-attachments/assets/2d5d2e4e-079b-45dd-9fad-d4fb0bd2ca7f" />
<img width="1766" height="800" alt="Screenshot from 2026-09-14 13-22-42" src="https://github.com/user-attachments/assets/2e854321-f606-4c48-bbd4-d91cf0f0d15c" />




> **Note:** Ensure that port `8080` is allowed in the AWS EC2 Security Group for external Swagger access.

---

### Installation Flow

```text
AWS EC2 Ubuntu 24.04
        |
        v
Clone Employee API
        |
        v
Install ScyllaDB
        |
        v
Create employee_db
        |
        v
Install Redis
        |
        v
Install Go
        |
        v
Configure Employee API
        |
        v
Configure Swagger
        |
        v
Run Database Migration
        |
        v
Build Employee API
        |
        v
Run Employee API
        |
        v
Health Check
        |
        v
Swagger Verification
```


---



# 5. Conclusion

The Employee API was successfully set up as a POC on an AWS EC2 Ubuntu 24.04 instance without Docker.
ScyllaDB and Redis were configured as the required backend services. Go was installed, Docker-based application configuration was updated for local services, database migration was executed, and the Employee API was built and started on port `8080`.
The application was verified using the health endpoint and Swagger documentation.

---

# 6. Contact Information

| Name |         Email Address             |
| ---- | ----------------------------------|
| Ritu | ritu.dogra.snaatak@mygurukulam.co |

---

# 7. References

| Resource                | Link                                             |
| ----------------------- | ------------------------------------------------ |
| Employee API Repository | https://github.com/OT-MICROSERVICES/employee-api |
| Go Documentation        | https://go.dev/doc/                              |
| ScyllaDB Documentation  | https://docs.scylladb.com/                       |
| Swagger Documentation   | https://swagger.io/docs/                         |
| golang-migrate          | https://github.com/golang-migrate/migrate        |

