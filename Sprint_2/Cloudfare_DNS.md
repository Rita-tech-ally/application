# POC Of Domain and DNS Setup for Frontend Application

<p align="center">
  <img width="90" height="auto" alt="dns-icon" src="https://img.icons8.com/fluency/96/domain.png" />
</p>

---

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| ------ | ---------- | ------- | ---------- | ---------- | ---------- |
| Ritu | 29/09/2026 | 1.0 | Liyakhat/Anirudh | Aman Raj | Sandeep Rawat/Ravindra |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Domain Setup](#3-domain-setup)
   - [3.1 Register/Get Domain](#31-registerget-domain)
   - [3.2 Create Route 53 Hosted Zone](#32-create-route-53-hosted-zone)
   - [3.3 Configure Name Servers / Delegation](#33-configure-name-servers--delegation)
4. [DNS Configuration](#4-dns-configuration)
   - [4.1 Create Subdomain](#41-create-subdomain)
   - [4.2 Create A Record](#42-create-a-record)
   - [4.3 Point A Record to EC2 Public IP](#43-point-a-record-to-ec2-public-ip)
5. [Configure Domain on Application](#5-configure-domain-on-application)
   - [5.1 Configure NGINX server_name](#51-configure-nginx-server_name)
   - [5.2 Configure Frontend](#52-configure-frontend)
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

For this POC, `sohandogra.com` is registered through Cloudflare and AWS Route 53 is used for DNS management.

The frontend application is already hosted on an EC2 instance using NGINX. The subdomain `ritu.sohandogra.com` is mapped to the EC2 public IP using an A record.

### Final Flow

```text
User Browser
     |
     v
ritu.sohandogra.com
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
| Cloudflare account | Domain registration |
| Registered domain | Base domain |
| AWS account | Route 53 and EC2 access |
| Route 53 access | DNS management |
| Running EC2 instance | Hosts the frontend |
| EC2 Public IP | A record target |
| NGINX | Serves the frontend |
| SSH access | NGINX configuration |
| Internet access | Validation |

### POC Environment

| Configuration | Value |
|---|---|
| Registered Domain | `sohandogra.com` |
| Domain Registrar | Cloudflare |
| DNS Service | AWS Route 53 |
| Application Subdomain | `ritu.sohandogra.com` |
| EC2 Instance | `ot-poc` |
| EC2 Public IP | `3.110.201.212` |
| Web Server | NGINX |
| HTTP Port | `80` |

---

# 3. Domain Setup

## 3.1 Register/Get Domain

A domain is required to provide a custom hostname for the application.

For this POC, the domain:

```text
sohandogra.com
```

was registered through Cloudflare.

The application uses:

```text
ritu.sohandogra.com
```

Here, `sohandogra.com` is the registered domain and `ritu` is the subdomain label.

### Steps

1. Log in to Cloudflare.
2. Search for the required domain.
3. Register the domain if it is not already registered.
4. Verify that the domain is active.

<!-- Screenshot: Cloudflare domain registration / domain overview -->

---

## 3.2 Create Route 53 Hosted Zone

Create a public hosted zone in AWS Route 53 for the registered domain.

### Steps

1. Log in to the AWS Management Console.
2. Open **Route 53**.
3. Select **Hosted zones**.
4. Click **Create hosted zone**.
5. Enter:

```text
sohandogra.com
```

6. Select **Public hosted zone**.
7. Click **Create hosted zone**.

Route 53 creates the hosted zone along with NS and SOA records.

<!-- Screenshot: Route 53 hosted zone -->

---

## 3.3 Configure Name Servers / Delegation

Since the domain is registered through Cloudflare and DNS is managed through Route 53, the Route 53 name servers need to be configured for DNS delegation.

### Steps

1. Open the Route 53 hosted zone.
2. Locate the **NS record**.
3. Copy the four Route 53 name servers.
4. Open the domain DNS/name-server settings in Cloudflare.
5. Configure the Route 53 name servers.
6. Save the changes.

### Delegation Flow

```text
Cloudflare
    |
    | Domain Registration
    v
sohandogra.com
    |
    | DNS Delegation
    v
AWS Route 53
```

<!-- Screenshot: Route 53 NS records -->

<!-- Screenshot: Cloudflare name-server configuration -->

---

# 4. DNS Configuration

## 4.1 Create Subdomain

The frontend application is accessed using:

```text
ritu.sohandogra.com
```

The subdomain label is:

```text
ritu
```

The registered domain is:

```text
sohandogra.com
```

Therefore:

```text
sohandogra.com
      |
      +---- ritu.sohandogra.com
```

This subdomain will be mapped to the EC2 public IP.

<!-- Screenshot: Route 53 hosted zone -->

---

## 4.2 Create A Record

An A record maps a hostname to an IPv4 address.

Create an A record for:

```text
ritu.sohandogra.com
```

### Record Configuration

| Configuration | Value |
|---|---|
| Record Name | `ritu` |
| Record Type | `A` |
| Routing Policy | `Simple` |
| Alias | `No` |
| Value | `3.110.201.212` |
| TTL | `60` seconds |

### Steps

1. Open the Route 53 hosted zone for `sohandogra.com`.
2. Click **Create record**.
3. Enter `ritu` as the record name.
4. Select record type `A`.
5. Enter `3.110.201.212` as the value.
6. Set TTL to `60` seconds.
7. Click **Create records**.

<!-- Screenshot: Route 53 Create A Record -->

---

## 4.3 Point A Record to EC2 Public IP

The A record maps the subdomain to the public IP of the EC2 instance.

```text
ritu.sohandogra.com
        |
        | A Record
        v
3.110.201.212
        |
        v
EC2 Instance: ot-poc
```

When the user accesses:

```text
http://ritu.sohandogra.com
```

DNS resolves the hostname to the EC2 public IP.

<!-- Screenshot: Route 53 A record -->

---

# 5. Configure Domain on Application

After configuring DNS, NGINX must be configured to respond to the domain.

The frontend is already hosted on the EC2 instance.

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
    server_name ritu.sohandogra.com;

    root /home/ubuntu/frontend/build;
    index index.html;

    location / {
        try_files $uri /index.html;
    }
}
```

The important directive is:

```nginx
server_name ritu.sohandogra.com;
```

This tells NGINX to handle requests for the configured domain.

The frontend is served from:

```text
/home/ubuntu/frontend/build
```

<!-- Screenshot: NGINX configuration -->

---

## 5.2 Configure Frontend

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
ritu.sohandogra.com
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
React Frontend
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
nslookup ritu.sohandogra.com
```

The result should contain:

```text
3.110.201.212
```

### Expected Result

```text
Name:    ritu.sohandogra.com
Address: 3.110.201.212
```

This confirms that the subdomain resolves to the EC2 public IP.

```text
ritu.sohandogra.com
        |
        v
3.110.201.212
```

<!-- Screenshot: nslookup result -->

---

## 6.2 Access Application Using Domain

Open the following URL in a browser:

```text
http://ritu.sohandogra.com
```

The browser resolves the domain through DNS and sends the request to the EC2 instance.

NGINX receives the request and serves the frontend application.

### Complete Request Flow

```text
User Browser
     |
     | http://ritu.sohandogra.com
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
React Frontend
```

### Expected Result

The frontend application opens successfully through:

```text
http://ritu.sohandogra.com
```

<!-- Screenshot: Application dashboard using domain -->

<!-- Screenshot: Browser address bar showing ritu.sohandogra.com -->

---

# 7. POC Result

The POC was completed successfully.

The domain `sohandogra.com` was registered through Cloudflare and DNS management was configured using AWS Route 53.

The subdomain:

```text
ritu.sohandogra.com
```

was configured with an A record pointing to the EC2 public IP:

```text
3.110.201.212
```

NGINX was configured with the same domain using the `server_name` directive.

DNS resolution was verified
