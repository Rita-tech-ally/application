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
   - [3.1 Register/Get Domain](#31-registerget-domain)
   - [3.2 Create Route 53 Hosted Zone](#32-create-route-53-hosted-zone)
   - [3.3 Configure Name Servers / Delegation](#33-configure-name-servers--delegation)
4. [DNS Configuration](#4-dns-configuration)
   - [4.1 Configure Domain](#41-configure-domain)
   - [4.2 Create A Record](#42-create-a-record)
   - [4.3 Point A Record to EC2 Public IP](#43-point-a-record-to-ec2-public-ip)
5. [Configure Domain on Application](#5-configure-domain-on-application)
   - [5.1 Configure NGINX server_name](#51-configure-nginx-server_name)
   - [5.2 Verify Frontend](#52-verify-frontend)
   - [5.3 Reload NGINX](#53-reload-nginx)
6. [DNS Validation](#6-dns-validation)
   - [6.1 Verify DNS Resolution](#61-verify-dns-resolution)
   - [6.2 Access Application Using Domain](#62-access-application-using-domain)
7. [POC Result](#7-poc-result)
8. [Contact Information](#8-contact-information)
9. [References](#9-references)

---

# 1. Introduction

This POC demonstrates how to configure a custom domain for a frontend application hosted on an AWS EC2 instance.

For this POC, the domain `devsecurity.shop` is registered through Hostinger and AWS Route 53 is used for DNS management.

The frontend application is already hosted on an EC2 instance using NGINX.

The domain `devsecurity.shop` is mapped directly to the EC2 public IP using an A record.

### Final Flow

```text
User Browser
     |
     v
devsecurity.shop
     |
     v
AWS Route 53
     |
     v
A Record
     |
     v
EC2 Public IP
     |
     v
NGINX
     |
     v
Frontend Application
```

---

# 2. Prerequisites

| Requirement | Purpose |
|---|---|
| Hostinger account | Domain registration |
| Registered domain | Application domain |
| AWS account | Route 53 and EC2 access |
| Route 53 access | DNS management |
| Running EC2 instance | Hosts the frontend |
| EC2 Public IP | A record target |
| NGINX | Serves the frontend |
| SSH access | NGINX configuration |
| Internet access | DNS and application validation |

### Domain Details

| Configuration | Value |
|---|---|
| Domain Name | `devsecurity.shop` |
| Domain Registrar | Hostinger |
| DNS Service | AWS Route 53 |
| EC2 Instance | `ot-poc` |
| EC2 Public IP | `3.110.201.212` |
| Web Server | NGINX |
| HTTP Port | `80` |

### Registered Domain Status

| Domain Name | Status | Expiration Date | Auto-Renewal |
|---|---|---|---|
| `devsecurity.shop` | Active | As shown in Hostinger | As configured in Hostinger |

---

# 3. Domain Setup

## 3.1 Register/Get Domain

A domain is required to provide a custom hostname for the frontend application.

For this POC, the domain is:

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

### Domain Information

```text
Domain:
devsecurity.shop

Registrar:
Hostinger

Status:
Active
```

<!-- Screenshot: Hostinger domain overview -->

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

<!-- Screenshot: Route 53 hosted zone -->

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

### Delegation Flow

```text
Hostinger
    |
    | Domain Registration
    v
devsecurity.shop
    |
    | DNS Delegation
    v
AWS Route 53
```

<!-- Screenshot: Route 53 NS records -->

<!-- Screenshot: Hostinger nameserver configuration -->

---

# 4. DNS Configuration

## 4.1 Configure Domain

The frontend application will be accessed directly using:

```text
devsecurity.shop
```

No separate application subdomain is used in this POC.

The domain is mapped directly to the EC2 public IP.

```text
devsecurity.shop
        |
        v
EC2 Public IP
```

---

## 4.2 Create A Record

An A record maps a domain name to an IPv4 address.

Create an A record for:

```text
devsecurity.shop
```

### Record Configuration

| Configuration | Value |
|---|---|
| Record Name | `@` |
| Record Type | `A` |
| Routing Policy | `Simple` |
| Alias | `No` |
| Value | `3.110.201.212` |
| TTL | `60` seconds |

### Steps

1. Open the Route 53 hosted zone for `devsecurity.shop`.
2. Click **Create record**.
3. Keep the record name empty for the root domain.
4. Select record type `A`.
5. Enter:

```text
3.110.201.212
```

6. Set TTL to `60` seconds.
7. Click **Create records**.

<!-- Screenshot: Route 53 Create A Record -->

---

## 4.3 Point A Record to EC2 Public IP

The A record connects the domain with the EC2 instance.

```text
devsecurity.shop
        |
        | A Record
        v
3.110.201.212
        |
        v
EC2 Instance: ot-poc
```

When a user accesses:

```text
http://devsecurity.shop
```

DNS resolves the domain to the EC2 public IP.

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

<!-- Screenshot: NGINX configuration -->

---

## 5.2 Verify Frontend

The frontend application is already available on the EC2 instance.

Verify the existing frontend build:

```bash
ls -la /home/ubuntu/frontend/build
```

The build directory should contain:

```text
index.html
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

<!-- Screenshot: Frontend build directory -->

---

## 5.3 Reload NGINX

Test the NGINX configuration:

```bash
sudo nginx -t
```

### Expected Result

```text
syntax is ok
test is successful
```

Reload NGINX:

```bash
sudo systemctl reload nginx
```

Verify the service:

```bash
sudo systemctl status nginx
```

### Expected Result

```text
active (running)
```

<!-- Screenshot: nginx -t result -->

<!-- Screenshot: NGINX service status -->

---

# 6. DNS Validation

After configuring the A record and NGINX, verify DNS resolution and application access.

---

## 6.1 Verify DNS Resolution

Run:

```bash
nslookup devsecurity.shop
```

The result should contain:

```text
3.110.201.212
```

### Expected Result

```text
Name:    devsecurity.shop
Address: 3.110.201.212
```

This confirms that the domain resolves to the EC2 public IP.

```text
devsecurity.shop
        |
        v
3.110.201.212
```

<!-- Screenshot: nslookup result -->

---

## 6.2 Access Application Using Domain

Open the following URL in a browser:

```text
http://devsecurity.shop
```

The browser resolves the domain through DNS and sends the request to the EC2 instance.

NGINX receives the request and serves the frontend application.

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
3.110.201.212
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

### Expected Result

The frontend application opens successfully through:

```text
http://devsecurity.shop
```

<!-- Screenshot: Application dashboard using domain -->

<!-- Screenshot: Browser address bar showing devsecurity.shop -->

---

# 7. POC Result

The POC was completed successfully.

The domain:

```text
devsecurity.shop
```

was registered through Hostinger.

AWS Route 53 was configured for DNS management.

The root domain was configured with an A record pointing to the EC2 public IP:

```text
3.110.201.212
```

NGINX was configured with the domain using the `server_name` directive.

DNS resolution was verified using `nslookup`, and the frontend application was successfully accessed using the custom domain.

### Final Architecture

```text
                    Hostinger
                Domain Registration
                         |
                         v
                  devsecurity.shop
                         |
                  DNS Delegation
                         |
                         v
                   AWS Route 53
                         |
                      A Record
                         |
                         v
                  3.110.201.212
                         |
                         v
                    AWS EC2
                    ot-poc
                         |
                         v
                      NGINX
                         |
                         v
                Frontend Application
```

### Final Configuration

| Component | Configuration |
|---|---|
| Registered Domain | `devsecurity.shop` |
| Domain Registrar | Hostinger |
| DNS Management | AWS Route 53 |
| DNS Record | A |
| Record Name | `@` |
| EC2 Public IP | `3.110.201.212` |
| Web Server | NGINX |
| Application | Frontend Application |
| Application URL | `http://devsecurity.shop` |

---

# 8. Contact Information

| Name | Email Address |
|---|---|
| Ritu | ritu.dogra.snaatak@mygurukulam.com |

---

# 9. References

| Reference | Description |
|---|---|
| AWS Route 53 Documentation | AWS DNS service documentation |
| Hostinger Documentation | Domain and DNS management documentation |
| NGINX Documentation | NGINX configuration reference |
| DNS Basics | DNS concepts and working |
| OT-Microservices Frontend | Frontend source repository |

---
