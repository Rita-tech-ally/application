# POC Of Domain and DNS Setup for Frontend Application

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
3. [Domain Setup](#3-domain-setup)
4. [DNS Configuration](#4-dns-configuration)
5. [Configure Domain on Application](#5-configure-domain-on-application)
6. [DNS Validation](#6-dns-validation)
7. [Conclusion](#7-conclusion)
8. [Contact Information](#8-contact-information)
9. [References](#9-references)

---

# 1. Introduction

This POC demonstrates how to get a custom domain and configure it for an existing frontend application hosted on an AWS EC2 instance.
In this POC, the domain `devsecurity.shop` is registered through Hostinger, and AWS Route 53 is used to manage DNS.
The domain is configured with an **A record** that points to the EC2 public IP. NGINX is then configured to serve the existing frontend application using the custom domain.

Finally, the domain is validated by accessing the frontend application through `http://devsecurity.shop`.

---

# 2. Prerequisites

| Requirement | Purpose |
|---|---|
| **Hostinger account** | Domain registration |
| **Registered domain** | Application domain |
| **AWS account** | Route 53 and EC2 access |
| **Route 53 access** | DNS management |
| **Running EC2 instance** | Hosts the frontend |
| **EC2 Public IP** | A record target |
| **NGINX** | Serves the frontend |
| **SSH access** | NGINX configuration |
| **Internet access** | DNS and application validation |

<img width="1593" height="583" alt="Screenshot from 2026-10-09 02-05-59" src="https://github.com/user-attachments/assets/12074397-1978-4731-8958-2850728fc88d" />

## 2.1 Configure Security Group

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

# 3. Domain Setup

## 3.1 Register/Get Domain

A domain is required to provide a custom hostname for the frontend application.

The domain is:

```text
devsecurity.shop
```

The domain was registered through Hostinger.

### Steps

1. Log in to Hostinger.
2. Open the domain registration section.
3. Search for the required domain.
4. Register `devsecurity.shop`.
5. Verify that the domain status is **Active**.
6. Confirm that the domain is available in the Hostinger domain management section.

<img width="1202" height="252" alt="Screenshot from 2026-10-09 01-38-00" src="https://github.com/user-attachments/assets/09077a08-a184-446c-a3d9-3435beea7d09" />

---

## 3.2 Create Route 53 Hosted Zone

AWS Route 53 is used to manage the DNS records for the domain.

### Steps

1. Log in to the AWS Management Console.
2. Open **Route 53**.
3. Select **Hosted zones**.
4. Click **Create hosted zone**.
5. Enter:

```text
devsecurity.shop
```

6. Select **Public hosted zone**.
7. Click **Create hosted zone**.

Route 53 creates the hosted zone with the required NS and SOA records.

<img width="1534" height="153" alt="Screenshot from 2026-10-09 01-40-14" src="https://github.com/user-attachments/assets/7932aa55-8643-4ccd-812b-7be606eea350" />

---

## 3.3 Configure Name Servers / Delegation

Since the domain is registered through Hostinger and DNS is managed using Route 53, the Route 53 name servers need to be configured in Hostinger.

### Steps

1. Open the Route 53 hosted zone.
2. Locate the **NS record**.
3. Copy the Route 53 name servers.
4. Open Hostinger.
5. Open the DNS or nameserver settings for `devsecurity.shop`.
6. Replace the existing nameservers with the Route 53 nameservers.
7. Save the changes.

<img width="1571" height="363" alt="Screenshot from 2026-10-09 01-41-10" src="https://github.com/user-attachments/assets/0eab5d88-4b88-4c42-a4d5-2a73a638fd9e" />

<img width="1161" height="329" alt="Screenshot from 2026-10-09 01-41-41" src="https://github.com/user-attachments/assets/450791a4-b096-4de9-8497-bb03734d0ee1" />

---

# 4. DNS Configuration

## 4.1 Create A Record

An A record maps a domain name to an IPv4 address.

Create an A record for:

```text
devsecurity.shop
```

### Record Configuration

| Configuration | Value |
|---|---|
| Record Name | *(blank, root domain)* |
| Record Type | `A` |
| Routing Policy | `Simple` |
| Alias | `No` |
| Value | `15.252.181.35` |
| TTL | `60` seconds |

### Steps

1. Open the Route 53 hosted zone for `devsecurity.shop`.
2. Click **Create record**.
3. Keep the record name empty for the root domain.
4. Select record type `A`.
5. Enter:

```text
15.252.181.35
```

6. Set TTL to `60` seconds.
7. Click **Create records**.

<img width="339" height="548" alt="Screenshot from 2026-10-09 01-42-43" src="https://github.com/user-attachments/assets/52085700-a7f1-4172-9a17-f2abd6bcfb84" />

---

# 5. Configure Domain on Application

After configuring DNS, NGINX needs to be configured to respond to the domain.

The frontend application is already hosted on the EC2 instance.

NGINX serves the existing frontend build.

---

## 5.1 Configure NGINX server_name

Connect to the EC2 instance and open the NGINX configuration:

```bash
sudo nano /etc/nginx/sites-available/ot-poc
```

Configure the server block:

```nginx
server {
    listen 80;
    server_name devsecurity.shop;

    root /home/ubuntu/frontend/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

The important directive is:

```nginx
server_name devsecurity.shop;
```

This tells NGINX to handle requests received for `devsecurity.shop`.

The frontend is served from:

```text
/home/ubuntu/frontend/build
```

---

## 5.2 Verify Frontend

The frontend application is already available on the EC2 instance.

Verify the existing frontend build:

```bash
ls -la /home/ubuntu/frontend/build
```

The application flow is:

```text
devsecurity.shop
        |
        v
NGINX
        |
        v
/home/ubuntu/frontend/build
        |
        v
index.html
        |
        v
Frontend Application
```

<img width="989" height="194" alt="Screenshot from 2026-10-09 01-57-45" src="https://github.com/user-attachments/assets/48bbaa41-672d-4767-bb7a-82f247d3cd59" />

---

## 5.3 Reload NGINX

Test the NGINX configuration:

```bash
sudo nginx -t
```

<img width="880" height="77" alt="image" src="https://github.com/user-attachments/assets/948a69d7-1f9c-4cc9-9844-1ed41eaa89b5" />

Reload NGINX:

```bash
sudo systemctl reload nginx
sudo systemctl status nginx
```

<img width="1298" height="566" alt="Screenshot from 2026-10-09 02-00-51" src="https://github.com/user-attachments/assets/8f15dd1c-c686-48a7-9ea4-1be1d95ad846" />

---

# 6. DNS Validation

After configuring the A record and NGINX, verify DNS resolution and application access.

---

## 6.1 Verify DNS Resolution

Run:

```bash
nslookup devsecurity.shop
```

**Expected Result:** The domain resolves to `15.252.181.35`.

<img width="1297" height="172" alt="Screenshot from 2026-10-09 02-01-35" src="https://github.com/user-attachments/assets/ccbf5f4c-1929-4b7f-89d2-5ce8b162a6eb" />

---

## 6.2 Access Application Using Domain

Open the following URL in a browser:

```text
http://devsecurity.shop
```

The browser resolves the domain through DNS and sends the request to the EC2 instance.

NGINX receives the request and serves the frontend application.

<img width="1773" height="972" alt="image" src="https://github.com/user-attachments/assets/bded158e-92a7-4d8a-9333-d19ee3931fb1" />

### Complete Request Flow

```text
User Browser
     |
     | http://devsecurity.shop
     v
AWS Route 53
     |
     | A Record
     v
15.252.181.35
     |
     v
AWS EC2
     |
     v
NGINX
     |
     v
Frontend Application
```

---

# 7. Conclusion

This POC successfully demonstrated how to get a custom domain and configure it for an existing frontend application.
The domain `devsecurity.shop` was registered through Hostinger, and AWS Route 53 was configured to manage its DNS.
An **A record** was created to point the domain to the EC2 public IP, and NGINX was configured to serve the existing frontend application using the custom domain.

The DNS configuration was validated, and the frontend application was successfully accessed using `http://devsecurity.shop`.

---

# 8. Contact Information

| Name | Email Address |
|---|---|
| Ritu | ritu.dogra.snaatak@mygurukulam.com |

---

# 9. References

| Reference | Description |
|---|---|
| **AWS Route 53 Documentation** | AWS DNS service documentation |
| **Hostinger Documentation** | Domain and DNS management documentation |
| **NGINX Documentation** | NGINX configuration reference |
| **DNS Basics** | DNS concepts and working |
| **OT-Microservices Frontend** | Frontend source repository |

---
