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
| Private IPv4        | `172.31.22.120`                               |
| Public IPv4         | `3.110.51.28`                                 |

> Attach an **Elastic IP** so the public IP does not change after a restart.

<img width="1645" height="564" alt="Screenshot from 2026-09-29 16-56-48" src="https://github.com/user-attachments/assets/3be87282-94d0-4790-9f30-89a74186683d" />


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

<img width="1129" height="128" alt="Screenshot from 2026-09-29 17-03-49" src="https://github.com/user-attachments/assets/0d8a211f-79ac-47b7-87ba-86ca7c5b2f85" />



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

📸 **Screenshot 7:** `ls -R /var/www/mock` output.

---

# 6. NGINX Configuration

## 6.1 Create the Configuration File

```bash
sudo nano /etc/nginx/sites-available/ot-poc
```

```nginx
server {
    listen 80 default_server;
    server_name devsecurity.shop www.devsecurity.shop _;

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

<img width="1124" height="134" alt="Screenshot from 2026-09-29 17-05-42" src="https://github.com/user-attachments/assets/1e2e3760-9e92-46b4-89eb-2d9c2d59ef3d" />

---

# 7. Validate Using Public IP

## 7.1 Local Checks on EC2

```bash
curl -I http://localhost
curl http://localhost/employee/search/all
```

**Expected Result:** `HTTP/1.1 200 OK` for the first command and the employee JSON for the second.

<img width="1138" height="329" alt="Screenshot from 2026-09-29 17-08-03" src="https://github.com/user-attachments/assets/31cfb6e1-f2f3-4a10-bfed-0fd71bbb3bc5" />


## 7.2 Browser Check

Open (type `http://` manually, SSL is not configured):

```text
http://3.110.51.28
```

**Expected Result:** The dashboard is displayed with stat cards (Total, Active, Ex-Employees, Office Locations) and the Role, Employee and Location donut charts. Employee List, Attendance List and Salary pages also show the mock data.

<img width="1655" height="707" alt="Screenshot from 2026-09-29 17-08-57" src="https://github.com/user-attachments/assets/42835884-f5a4-4bb0-ad0c-c5671e7799a5" />

<img width="1644" height="653" alt="Screenshot from 2026-09-29 17-09-26" src="https://github.com/user-attachments/assets/8747f315-a30f-4c40-b661-efadc4d68f17" />


---

# 8. Domain and DNS Setup (Route 53)

## 8.1 Domain and Hosted Zone

The domain `devsecurity.shop` is managed in **AWS Route 53**. The public hosted zone contains the default NS and SOA records:

| **Record Type** | **Value** |
| --------------- | --------- |
| NS | `ns-67.awsdns-08.com.` <br> `ns-1910.awsdns-46.co.uk.` <br> `ns-866.awsdns-44.net.` <br> `ns-1530.awsdns-63.org.` |

📸 **Screenshot 12:** Route 53, Hosted zones, `devsecurity.shop` (NS and SOA records).

## 8.2 Create A Record

Route 53, Hosted zones, `devsecurity.shop`, **Create record**:

| **Configuration** | **Value**                                    |
| ----------------- | -------------------------------------------- |
| Record Name       | *(leave blank for root domain)*              |
| Record Type       | `A`                                          |
| Routing Policy    | `Simple`                                     |
| Alias             | `No`                                         |
| Value             | `43.204.227.47` (EC2 Elastic IP)               |
| TTL               | `300` seconds                                |

> **Note:** Leave the record name blank for the root domain. Typing the full domain in the name field creates `devsecurity.shop.devsecurity.shop`, which is incorrect.

Optional: create another A record with name `www` and the same IP for `www.devsecurity.shop`.

<img width="1165" height="321" alt="Screenshot from 2026-09-29 17-11-34" src="https://github.com/user-attachments/assets/58ccd875-980e-43cd-ba4f-04c11b41bed5" />


---

# 9. DNS Validation

## 9.1 Verify DNS Resolution

```bash
nslookup devsecurity.shop
```

**Expected Result:** The domain resolves to the EC2 public IP.

```text
Name:   devsecurity.shop
Address: 3.110.51.28
```

📸 **Screenshot 14:** `nslookup` output.

## 9.2 Access the Application Using the Domain

```text
http://devsecurity.shop
```

📸 **Screenshot 15:** Dashboard opened using `http://devsecurity.shop` (domain visible in the address bar).

---

# 10. POC Result

The POC was completed successfully. The React frontend is hosted on an AWS EC2 instance with NGINX, and the dashboard shows data through static mock JSON, without any backend service. The domain `devsecurity.shop` is mapped to the EC2 public IP using an **A record in AWS Route 53**.

### Final Request Flow

```text
User Browser
     |
     v
devsecurity.shop
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

### Limitations

* **Add Employee** and **Add Attendance** forms do not work, because they send POST requests and static files do not accept POST (405).
* The data is fixed sample data. To change it, edit the JSON files in `/var/www/mock`. No rebuild is needed.
* The site runs on HTTP. HTTPS can be added later with Certbot.

---

# 11. Issues Faced and Fixes

| **Issue** | **Cause** | **Fix** |
| --------- | --------- | ------- |
| `ERR_OSSL_EVP_UNSUPPORTED` during build | Old webpack with Node 18 | Use Node 16 through nvm |
| `heap out of memory` / `ENOSPC` | Small instance and full disk | Larger instance, 20 GB disk, swap file |
| `500 Internal Server Error` (`rewrite or internal redirection cycle`) | NGINX `root` had no `index.html` | Point `root` to `/home/ubuntu/frontend/build` |
| `403 / 500` for build files | Home directory permission on Ubuntu 24.04 | `chmod o+x /home/ubuntu` and `chmod -R o+rX build` |
| Domain not resolving | A record created as `devsecurity.shop.devsecurity.shop` | Recreate record with blank name |
| `nginx -t` failed (`sites-enabled/ot-poc` missing) | Symlink created without the config file | Create the file in `sites-available`, then relink |

---

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
