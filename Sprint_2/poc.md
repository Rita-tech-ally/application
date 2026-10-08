# POC Of Frontend Hosting with DNS

<p align="center">
  <img width="90" height="auto" alt="dns-icon" src="https://img.icons8.com/fluency/96/domain.png" />
</p>

---

## Document Information

| Author | Created On | Version | L0 Reviewer      | L1 Reviewer | L2 Reviewer            |
| ------ | ---------- | ------- | ---------------- | ----------- | ---------------------- |
| Ritu   | 29/09/2026 | 1.0     | Liyakhat/Anirudh | Aman Raj    | Sandeep Rawat/Ravindra |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Application Setup](#3-application-setup)
4. [Frontend Setup](#4-frontend-setup)
5. [Validate Using Public IP](#5-validate-using-public-ip)
6. [Domain and DNS Setup](#6-domain-and-dns-setup)
7. [DNS Validation](#7-dns-validation)
8. [POC Result](#8-poc-result)
9. [Contact Information](#9-contact-information)
10. [References](#10-references)

---

# 1. Introduction

This document demonstrates hosting the OT-Microservices React frontend on an AWS EC2 instance and accessing it through a custom domain, `ritu.sohandogra.com`, mapped using AWS Route 53.

---

# 2. Prerequisites

The following are required to perform this POC:

* AWS account
* AWS EC2 instance (Ubuntu 24.04 LTS)
* SSH private key
* AWS Route 53 access
* Cloudflare account with a registered domain
* Internet access
* Git
* Node.js 16 (installed using nvm)
* NGINX

---

# 3. Application Setup

## 3.1 Create EC2 Instance

Create an Ubuntu EC2 instance in AWS.

The following EC2 instance was used for this POC:

| **Configuration**    | **Value**              |
| -------------------- | ---------------------- |
| **Instance Name**    | `ot-poc`               |
| **Instance Type**    | `t3.micro` |
| **Operating System** | Ubuntu 24.04.4 LTS     |
| **Private IPv4**     | `172.31.30.166`        |
| **Public IPv4**      | `3.110.201.212`        |

After the instance is running, note the **Public IPv4 address** because it will be used in the Route 53 A record.


<img width="1302" height="542" alt="Screenshot from 2026-09-29 17-57-40" src="https://github.com/user-attachments/assets/523e180d-a2dd-46f8-af2a-9744a203ff3d" />

---

## 3.2 Configure Security Group

Configure the EC2 Security Group to allow the required traffic.

| **Type**  | **Port** | **Source**      |
| --------- | -------: | --------------- |
| **SSH**   |       22 | Your IP address |
| **HTTP**  |       80 | `0.0.0.0/0`     |
| **HTTPS** |      443 | `0.0.0.0/0`     |

**Purpose:**

* **Port 22** – Allows SSH access to the EC2 instance.
* **Port 80** – Allows HTTP traffic to reach NGINX.
* **Port 443** – Allows HTTPS traffic if SSL/TLS is configured later.

---

# 4. Frontend Setup

## 4.1 Install Node.js 16 using nvm

The frontend uses `react-scripts 2.x`, which builds without OpenSSL errors on Node 16.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.nvm/nvm.sh
nvm install 16
nvm alias default 16
node -v
```

<img width="596" height="74" alt="Screenshot from 2026-10-01 10-25-48" src="https://github.com/user-attachments/assets/287aa02a-36ec-498e-a5d6-375de469429f" />

## 4.2 Install NGINX

```bash
sudo apt update
sudo apt install -y nginx
sudo systemctl status nginx
```

**Expected Result:** NGINX shows `active (running)`.

<img width="1172" height="275" alt="Screenshot from 2026-09-29 19-54-46" src="https://github.com/user-attachments/assets/37fe4c7b-3afd-4f93-9b6d-b1d17e57a2d0" />

## 4.3 Clone the Frontend Repository

```bash
cd ~
git clone https://github.com/OT-MICROSERVICES/frontend.git
cd frontend
```

<img width="745" height="216" alt="Screenshot from 2026-09-30 14-12-02" src="https://github.com/user-attachments/assets/acd9a15d-5790-4e12-9574-d9df8e4f036a" />

## 4.4 Install Dependencies and Build

```bash
npm install --legacy-peer-deps
export NODE_OPTIONS=--max-old-space-size=2048
npm run build
ls build/index.html
```
---

## 4.5 Mock Data Setup

The backend services are not deployed in this POC. NGINX serves static JSON files on the same paths the frontend calls, so the dashboard can display data.

| **Frontend Request**        | **Mock File**                                 |
| --------------------------- | --------------------------------------------- |
| `/employee/search/all`      | `/var/www/mock/employee/search/all.json`      |
| `/employee/search/status`   | `/var/www/mock/employee/search/status.json`   |
| `/employee/search/roles`    | `/var/www/mock/employee/search/roles.json`    |
| `/employee/search/location` | `/var/www/mock/employee/search/location.json` |
| `/attendance/search`        | `/var/www/mock/attendance/search.json`        |
| `/salary/search/all`        | `/var/www/mock/salary/search/all.json`        |

**Create the folders:**

```bash
sudo mkdir -p /var/www/mock/{employee/search,attendance,salary/search}
cd /var/www/mock
```

**Create the JSON files:**

```bash
sudo tee employee/search/all.json >/dev/null <<'EOF'
[
 {"id":"OT-001","name":"Rahul Sharma","email":"rahul@example.com","phone_number":"9999990001","job_role":"DevOps","location":"Delhi"},
 {"id":"OT-002","name":"Priya Singh","email":"priya@example.com","phone_number":"9999990002","job_role":"Developer","location":"Bangalore"},
 {"id":"OT-003","name":"Amit Verma","email":"amit@example.com","phone_number":"9999990003","job_role":"DevOps","location":"Hyderabad"},
 {"id":"OT-004","name":"Neha Gupta","email":"neha@example.com","phone_number":"9999990004","job_role":"Developer","location":"Newyork"},
 {"id":"OT-005","name":"Karan Mehta","email":"karan@example.com","phone_number":"9999990005","job_role":"Developer","location":"Delhi"}
]
EOF

echo '{"Current Employee":4,"Ex-Employee":1}' | sudo tee employee/search/status.json >/dev/null
echo '{"DevOps":2,"Developer":3}' | sudo tee employee/search/roles.json >/dev/null
echo '{"Delhi":2,"Bangalore":1,"Hyderabad":1,"Newyork":1}' | sudo tee employee/search/location.json >/dev/null

echo '[{"id":"OT-001","status":"Present","date":"2026-09-29"},{"id":"OT-002","status":"Absent","date":"2026-09-29"},{"id":"OT-003","status":"Present","date":"2026-09-29"}]' | sudo tee attendance/search.json >/dev/null

echo '[{"id":"OT-001","name":"Rahul Sharma","annual_package":1200000},{"id":"OT-002","name":"Priya Singh","annual_package":1000000},{"id":"OT-003","name":"Amit Verma","annual_package":1400000}]' | sudo tee salary/search/all.json >/dev/null
```

**Set permissions and verify:**

```bash
sudo chmod -R o+rX /var/www/mock
ls -R /var/www/mock
```

---

## 4.6 NGINX Configuration

**Create the configuration file:**

```bash
sudo nano /etc/nginx/sites-available/ot-poc
```

```nginx
server {
    listen 80 default_server;
    server_name ritu.sohandogra.com;

    root /home/ubuntu/frontend/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location ~ ^/(employee|attendance|salary)/ {
        root /var/www/mock;
        default_type application/json;
        add_header Cache-Control "no-store";
        try_files $uri.json =404;
    }
}
```

## 4.7 Enable the Configuration and Set Permissions

```bash
sudo ln -sf /etc/nginx/sites-available/ot-poc /etc/nginx/sites-enabled/ot-poc
sudo rm -f /etc/nginx/sites-enabled/default
chmod o+x /home/ubuntu
chmod -R o+rX /home/ubuntu/frontend/build
sudo nginx -t && sudo systemctl reload nginx
```

> `chmod o+x /home/ubuntu` is required on Ubuntu 24.04 because the home directory is not readable by other users by default. Without it NGINX returns 403/500.

**Expected Result:**

<img width="1073" height="69" alt="Screenshot from 2026-09-29 18-01-20" src="https://github.com/user-attachments/assets/d62e82c3-c491-4d9d-ac51-abd310ac0bab" />

---

# 5. Validate Using Public IP

## 5.1 Local Checks on EC2

```bash
curl -I http://localhost
curl http://localhost/employee/search/all
```

<img width="643" height="173" alt="Screenshot from 2026-09-29 19-21-51" src="https://github.com/user-attachments/assets/38eb6b42-4b77-4c50-adf4-89d5ddda6506" />


---

## 5.2 Browser Check

```text
http://3.110.201.212
```

**Expected Result:** The dashboard is displayed with stat cards (Total, Active, Ex-Employees, Office Locations) and the Role, Employee and Location donut charts. The Employee List, Attendance List and Salary pages also show the mock data.

<img width="1417" height="883" alt="Screenshot from 2026-09-29 18-03-10" src="https://github.com/user-attachments/assets/8ff7bb01-042f-4c21-a103-bd7f85f0bd62" />
<img width="1478" height="618" alt="Screenshot from 2026-09-29 18-03-34" src="https://github.com/user-attachments/assets/cbcf4642-ef73-4b05-9729-4ce02733e60e" />
<img width="1478" height="618" alt="Screenshot from 2026-09-29 18-03-50" src="https://github.com/user-attachments/assets/54105a43-0e30-4d06-ad15-bf4b7313d815" />
<img width="1490" height="500" alt="Screenshot from 2026-09-29 18-04-14" src="https://github.com/user-attachments/assets/ea9a2b67-fe70-46a1-8c91-0a5bd9fae86a" />

---

# 6. Domain and DNS Setup

## 6.1 Domain Details

The domain `sohandogra.com` is registered through **Cloudflare**.

The hostname used for this POC is:

```text
ritu.sohandogra.com
```

Here, `ritu.sohandogra.com` is a **subdomain** of the registered domain `sohandogra.com`.

```text
Registered Domain:
sohandogra.com

Subdomain:
ritu.sohandogra.com
```

Cloudflare is used for the **domain registration**, while AWS Route 53 is used for the **DNS management** of the subdomain in this POC.

---


## 6.2 Create A Record

Route 53, Hosted zones, `ritu.sohandogra.com`, **Create record**:

| **Configuration** | **Value**                                   |
| ----------------- | ------------------------------------------- |
| Record Name       | *(leave blank, the hosted zone name is added automatically)* |
| Record Type       | `A`                                         |
| Routing Policy    | `Simple`                                    |
| Alias             | `No`                                        |
| Value             | `3.110.201.212`                             |
| TTL               | `60` seconds                                |

<img width="1517" height="354" alt="Screenshot from 2026-09-29 18-05-25" src="https://github.com/user-attachments/assets/c0ebbe26-1d0d-47b0-ba4b-bd10d74a4493" />

---

# 7. DNS Validation

## 7.1 Verify DNS Resolution

```bash
nslookup ritu.sohandogra.com
```

**Expected Result:** The domain resolves to the EC2 public IP.

<img width="707" height="191" alt="Screenshot from 2026-09-29 18-06-39" src="https://github.com/user-attachments/assets/b076d66a-7cdd-4c33-b251-7524f6b0d14e" />

---

## 7.2 Access the Application Using the Domain


```text
http://ritu.sohandogra.com
```

**Expected Result:** The dashboard, Employee List, Attendance List and Salary pages open through the domain.

<img width="1920" height="1080" alt="dashoard_fr" src="https://github.com/user-attachments/assets/622cecbd-1b4e-49de-a6ad-e4bef5afa77c" />
<img width="1920" height="1080" alt="employ" src="https://github.com/user-attachments/assets/29a46535-7412-4851-8a89-b237806a5bac" />
<img width="1920" height="1080" alt="salary" src="https://github.com/user-attachments/assets/67bfb577-5a97-4043-b0ab-fcbb6a7b11c5" />
<img width="1920" height="1080" alt="salary" src="https://github.com/user-attachments/assets/a2f0716c-3fe6-4a9f-bb8c-5734b71617e0" />
<img width="1920" height="1080" alt="SALARY" src="https://github.com/user-attachments/assets/6812d96e-e1bf-4fad-9cbc-76a02a1d62ff" />

---

# 8. POC Result

The POC was completed successfully. The React frontend is hosted on an AWS EC2 instance with NGINX, and the dashboard shows data through static mock JSON, without any backend service. The subdomain `ritu.sohandogra.com` is mapped to the EC2 public IP using an A record in AWS Route 53, with the subdomain delegated from Cloudflare to Route 53 through NS records.

### Final Request Flow

```text
User Browser
     |
     v
ritu.sohandogra.com
     |
     v
Cloudflare (domain registrar, NS delegation for "ritu")
     |
     v
AWS Route 53 (A Record)
     |
     v
EC2 Public IP (3.110.201.212)
     |
     v
NGINX
     |---------------------------------------|
     v                                       v
React Build                         Mock JSON files
(/home/ubuntu/frontend/build)       (/var/www/mock)
Dashboard UI                        /employee, /attendance, /salary
```

---

# 9. Contact Information

| Name | Email Address                                                                 |
| ---- | ----------------------------------------------------------------------------- |
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.com) |

---

# 10. References

| **Reference**                                                              | **Description**                    |
| -------------------------------------------------------------------------- | ---------------------------------- |
| [OT-Microservices Frontend](https://github.com/OT-MICROSERVICES/frontend)  | Frontend source repository         |
| [AWS Route 53](https://aws.amazon.com/route53/)                            | Managed DNS service by AWS         |
| [NGINX Documentation](https://nginx.org/en/docs/)                          | NGINX configuration reference      |
| [DNS Basics](https://www.cloudflare.com/learning/dns/what-is-dns/)         | Basic explanation of how DNS works |
