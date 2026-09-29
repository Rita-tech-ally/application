# POC Of Frontend Deployment with DNS (Route 53)

---

## Document Information

| Author | Created On | Version | L0 Reviewer      | L1 Reviewer | L2 Reviewer            |
| ------ | ---------- | ------- | ---------------- | ----------- | ---------------------- |
| Ritu   | 29/09/2026 | 1.0     | Liyakhat/Anirudh | Aman Raj    | Sandeep Rawat/Ravindra |

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [EC2 Setup](#3-ec2-setup)
4. [Frontend Setup](#4-frontend-setup)
5. [Mock Data Setup](#5-mock-data-setup)
6. [NGINX Configuration](#6-nginx-configuration)
7. [Validate Using Public IP](#7-validate-using-public-ip)
8. [Domain and DNS Setup (Route 53)](#8-domain-and-dns-setup-route-53)
9. [DNS Validation](#9-dns-validation)
10. [POC Result](#10-poc-result)
11. [Issues Faced and Fixes](#11-issues-faced-and-fixes)
12. [Contact Information](#12-contact-information)
13. [References](#13-references)

---

# 1. Introduction

This document demonstrates hosting the **OT-Microservices React frontend** on an AWS EC2 instance and accessing it through a custom domain (`devsecurity.shop`) mapped using **AWS Route 53**.

The backend services (Employee API, Attendance API, Salary API) are **not deployed** in this POC. Instead, NGINX serves **static mock JSON files** on the same paths the frontend calls (`/employee/...`, `/attendance/...`, `/salary/...`). This allows the dashboard to display data without running any backend service or database.

---

# 2. Prerequisites

* AWS account
* AWS EC2 instance (Ubuntu 24.04 LTS)
* SSH private key (`.pem`)
* AWS Route 53 access (hosted zone for the domain)
* Registered domain: `devsecurity.shop`
* Internet access
* Git, Node.js 16, NGINX

---

# 3. EC2 Setup

## 3.1 Create EC2 Instance

| **Configuration**   | **Value**                                     |
| ------------------- | --------------------------------------------- |
| Instance Name       | `ot-poc`                                      |
| Operating System    | Ubuntu 24.04.4 LTS                            |
| Instance Type       | `<instance type used>` (4 GB+ RAM recommended) |
| Storage             | 20 GB gp3 recommended                         |
| Private IPv4        | `172.31.30.166`                               |
| Public IPv4         | `3.110.51.28`                                 |




<img width="1302" height="542" alt="Screenshot from 2026-09-29 17-57-40" src="https://github.com/user-attachments/assets/523e180d-a2dd-46f8-af2a-9744a203ff3d" />

---

## 3.2 Configure Security Group

| **Type** | **Port** | **Source**      |
| -------- | -------: | --------------- |
| SSH      |       22 | Your IP address |
| HTTP     |       80 | `0.0.0.0/0`     |
| HTTPS    |      443 | `0.0.0.0/0`     |



## 3.3 Connect and Install Packages


# 4. Frontend Setup

## 4.1 Install Node.js 16 using nvm

The frontend uses `react-scripts 2.x`, which works with Node 16 without OpenSSL errors.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.nvm/nvm.sh
nvm install 16
nvm alias default 16
node -v
```

**Expected Result:** `v16.20.2`

<img width="1008" height="64" alt="Screenshot from 2026-09-29 17-59-20" src="https://github.com/user-attachments/assets/e13a3a28-e11c-4711-88de-359134b1e38b" />




## 4.2 Clone the Frontend Repository

```bash
cd ~
git clone https://github.com/OT-MICROSERVICES/frontend.git
cd frontend
ls
```


## 4.3 Install Dependencies and Build

```bash
cd ~/frontend
npm install --legacy-peer-deps
export NODE_OPTIONS=--max-old-space-size=2048
npm run build
ls build/index.html
```

---

# 5. Mock Data Setup

The frontend calls these paths, so a matching JSON file is created for each:

| **Frontend Request**            | **Mock File**                                   |
| ------------------------------- | ----------------------------------------------- |
| `/employee/search/all`          | `/var/www/mock/employee/search/all.json`        |
| `/employee/search/status`       | `/var/www/mock/employee/search/status.json`     |
| `/employee/search/roles`        | `/var/www/mock/employee/search/roles.json`      |
| `/employee/search/location`     | `/var/www/mock/employee/search/location.json`   |
| `/attendance/search`            | `/var/www/mock/attendance/search.json`          |
| `/salary/search/all`            | `/var/www/mock/salary/search/all.json`          |

```bash
sudo mkdir -p /var/www/mock/{employee/search,attendance,salary/search}
cd /var/www/mock

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

sudo chmod -R o+rX /var/www/mock
ls -R /var/www/mock
```

<img width="1567" height="329" alt="Screenshot from 2026-09-29 18-00-32" src="https://github.com/user-attachments/assets/063d7598-6104-4137-99ef-98643d4ef100" />


---

# 6. NGINX Configuration

## 6.1 Create the Configuration File

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

**Explanation:**

* `location /` serves the React build and falls back to `index.html` for client-side routes.
* The `location ~ ^/(employee|attendance|salary)/` block serves the static JSON files instead of a backend API.

## 6.2 Enable the Configuration and Set Permissions

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

# 7. Validate Using Public IP

## 7.1 Local Checks on EC2

```bash
curl -I http://localhost
```

**Expected Result:** `HTTP/1.1 200 OK` for the first command and the employee JSON for the second.

<img width="643" height="173" alt="Screenshot from 2026-09-29 19-21-51" src="https://github.com/user-attachments/assets/38eb6b42-4b77-4c50-adf4-89d5ddda6506" />




## 7.2 Browser Check

Open (type `http://` manually, SSL is not configured):

```text
http://3.110.51.28
```

**Expected Result:** The dashboard is displayed with stat cards (Total, Active, Ex-Employees, Office Locations) and the Role, Employee and Location donut charts. Employee List, Attendance List and Salary pages also show the mock data.

<img width="1417" height="883" alt="Screenshot from 2026-09-29 18-03-10" src="https://github.com/user-attachments/assets/8ff7bb01-042f-4c21-a103-bd7f85f0bd62" />
<img width="1478" height="618" alt="Screenshot from 2026-09-29 18-03-34" src="https://github.com/user-attachments/assets/cbcf4642-ef73-4b05-9729-4ce02733e60e" />
<img width="1478" height="618" alt="Screenshot from 2026-09-29 18-03-50" src="https://github.com/user-attachments/assets/54105a43-0e30-4d06-ad15-bf4b7313d815" />
<img width="1490" height="500" alt="Screenshot from 2026-09-29 18-04-14" src="https://github.com/user-attachments/assets/ea9a2b67-fe70-46a1-8c91-0a5bd9fae86a" />


---

# 8. Domain and DNS Setup (Route 53)

## 8.1 Create A Record

Route 53, Hosted zones, `ritu.sohandogra.com`, **Create record**:

| **Configuration** | **Value**                                    |
| ----------------- | -------------------------------------------- |
| Record Name       | *(leave blank for root domain)*              |
| Record Type       | `A`                                          |
| Routing Policy    | `Simple`                                     |
| Alias             | `No`                                         |
| Value             | `3.110.51.28` (EC2 Elastic IP)               |
| TTL               | `60` seconds                                |

<img width="1517" height="354" alt="Screenshot from 2026-09-29 18-05-25" src="https://github.com/user-attachments/assets/c0ebbe26-1d0d-47b0-ba4b-bd10d74a4493" />


---

# 9. DNS Validation

## 9.1 Verify DNS Resolution

```bash
nslookup ritu.sohandogra.com
```

**Expected Result:** The domain resolves to the EC2 public IP.

<img width="707" height="191" alt="Screenshot from 2026-09-29 18-06-39" src="https://github.com/user-attachments/assets/b076d66a-7cdd-4c33-b251-7524f6b0d14e" />



## 9.2 Access the Application Using the Domain

```text
http://ritu.sohandogra.com
```

<img width="1920" height="1080" alt="dasboard" src="https://github.com/user-attachments/assets/3e0e6213-b1a5-482c-a73f-579fe7a08e47" />
<img width="1920" height="1080" alt="employ" src="https://github.com/user-attachments/assets/29a46535-7412-4851-8a89-b237806a5bac" />
<img width="1920" height="1080" alt="SALARY" src="https://github.com/user-attachments/assets/6812d96e-e1bf-4fad-9cbc-76a02a1d62ff" />

---

# 10. POC Result

The POC was completed successfully. The React frontend is hosted on an AWS EC2 instance with NGINX, and the dashboard shows data through static mock JSON, without any backend service. The domain `devsecurity.shop` is mapped to the EC2 public IP using an **A record in AWS Route 53**.

### Final Request Flow

```text
User Browser
     |
     v
ritu.sohandogra.com
     |
     v
AWS Route 53 (A Record)
     |
     v
EC2 Public / Elastic IP
     |
     v
NGINX
     |------------------------------------|
     v                                    v
React Build (/home/ubuntu/frontend/build)   Mock JSON (/var/www/mock)
 (dashboard UI)                             (/employee, /attendance, /salary)
```



# 12. Contact Information

| Name | Email Address                                                                 |
| ---- | ----------------------------------------------------------------------------- |
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

---

# 13. References

| **Reference** | **Description** |
| ------------- | --------------------------------------------- |
| [OT-Microservices Frontend](https://github.com/OT-MICROSERVICES/frontend) | Frontend source repository |
| [AWS Route 53](https://aws.amazon.com/route53/) | Managed DNS service by AWS |
| [NGINX Documentation](https://nginx.org/en/docs/) | NGINX configuration reference |
| [DNS Basics](https://www.cloudflare.com/learning/dns/what-is-dns/) | Basic explanation of how DNS works |

---
