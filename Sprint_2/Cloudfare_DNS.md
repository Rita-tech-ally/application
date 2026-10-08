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

This document demonstrates how to configure a custom domain for a frontend application hosted on an AWS EC2 instance.

In this POC, the domain `sohandogra.com` is registered through Cloudflare, while AWS Route 53 is used to manage DNS records for the application subdomain.

The frontend application is already hosted on an EC2 instance using NGINX. The purpose of this POC is to configure the domain `ritu.sohandogra.com`, point it to the EC2 public IP using a DNS A record, and configure NGINX to serve the frontend using the domain name.

### Final Flow

```text
User Browser
     |
     v
ritu.sohandogra.com
     |
     v
DNS Resolution
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

The following are required for this POC:

| Requirement | Purpose |
|---|---|
| Cloudflare account | Domain registration |
| Registered domain | Base domain for the application |
| AWS account | Route 53 and EC2 access |
| AWS Route 53 access | DNS management |
| Running EC2 instance | Hosts the frontend application |
| NGINX | Serves the frontend application |
| EC2 Public IP | Target for the DNS A record |
| SSH access | Configure NGINX on EC2 |
| Internet access | Verify domain resolution and application access |

### POC Details

| Configuration | Value |
|---|---|
| Registered Domain | `sohandogra.com` |
| Domain Registrar | Cloudflare |
| Application Subdomain | `ritu.sohandogra.com` |
| DNS Service | AWS Route 53 |
| EC2 Instance | `ot-poc` |
| EC2 Public IP | `3.110.201.212` |
| Web Server | NGINX |
| Application Protocol | HTTP |
| HTTP Port | `80` |

---

# 3. Domain Setup

## 3.1 Register/Get Domain

A domain is required before configuring DNS for the application.

For this POC, the domain:

```text
sohandogra.com
```

was registered through Cloudflare.

The registered domain is used as the base domain, and the application is exposed using the following subdomain:

```text
ritu.sohandogra.com
```

Here:

```text
sohandogra.com
      |
      +---- ritu.sohandogra.com
```

`sohandogra.com` is the registered domain, while `ritu.sohandogra.com` is the subdomain used for the frontend application.

### Steps

1. Log in to the Cloudflare account.
2. Search for the required domain.
3. Register the domain if it is not already registered.
4. Verify that the domain is active.
5. Use the registered domain as the base domain for DNS configuration.

<!-- Screenshot: Cloudflare domain registration / domain overview -->

---

## 3.2 Create Route 53 Hosted Zone

After obtaining the domain, create a hosted zone in AWS Route 53.

### Steps

1. Log in to the AWS Management Console.
2. Open **Route 53**.
3. Select **Hosted zones**.
4. Click **Create hosted zone**.
5. Enter the domain name:

```text
sohandogra.com
```

6. Select:

```text
Type: Public hosted zone
```

7. Click **Create hosted zone**.

Route 53 creates the hosted zone and provides a set of name servers.

These name servers are required to delegate DNS management to Route 53.

### Expected Result

A Route 53 hosted zone is created for:

```text
sohandogra.com
```

<!-- Screenshot: Route 53 hosted zone creation -->

<!-- Screenshot: Route 53 hosted zone showing NS and SOA records -->

---

## 3.3 Configure Name Servers / Delegation

The domain is registered through Cloudflare, but DNS management for this POC is handled by AWS Route 53.

Therefore, the Route 53 name servers need to be configured for the required DNS delegation.

Route 53 provides four name servers similar to:

```text
ns-xxx.awsdns-xx.org
ns-xxx.awsdns-xx.com
ns-xxx.awsdns-xx.net
ns-xxx.awsdns-xx.co.uk
```

### Steps

1. Open the Route 53 hosted zone.
2. Copy the four Route 53 name servers from the **NS record**.
3. Open the domain's DNS/name-server settings in Cloudflare.
4. Configure the required name-server delegation according to the DNS setup.
5. Save the changes.

After delegation, DNS queries for the configured domain/subdomain can be resolved through Route 53.

<!-- Screenshot: Route 53 NS records -->

<!-- Screenshot: Cloudflare DNS / nameserver configuration -->

### DNS Delegation Flow

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
    |
    v
DNS Records
```

---

# 4. DNS Configuration

## 4.1 Create Subdomain

The frontend application is accessed using:

```text
ritu.sohandogra.com
```

Here:

```text
ritu
```

is the subdomain label and:

```text
sohandogra.com
```

is the registered domain.

Therefore:

```text
ritu.sohandogra.com
```

is the complete hostname used by the application.

The subdomain is configured in the Route 53 hosted zone.

<!-- Screenshot: Route 53 hosted zone -->

---

## 4.2 Create A Record

An **A record** maps a domain or subdomain to an IPv4 address.

For this POC, the A record maps:

```text
ritu.sohandogra.com
```

to the EC2 public IPv4 address:

```text
3.110.201.212
```

### Steps

1. Open **AWS Route 53**.
2. Open the hosted zone for:

```text
sohandogra.com
```

3. Click **Create record**.
4. Configure the record as follows:

| Configuration | Value |
|---|---|
| Record Name | `ritu` |
| Record Type | `A` |
| Routing Policy | `Simple` |
| Alias | `No` |
| Value | `3.110.201.212` |
| TTL | `60` seconds |

5. Click **Create records**.

### Result

The DNS mapping becomes:

```text
ritu.sohandogra.com
        |
        v
3.110.201.212
```

<!-- Screenshot: Create A record in Route 53 -->

---

## 4.3 Point A Record to EC2 Public IP

The A record points the application subdomain to the public IPv4 address of the EC2 instance.

For this POC:

```text
Domain:
ritu.sohandogra.com

        ↓

A Record:
3.110.201.212

        ↓

EC2 Instance:
ot-poc
```

This means when a user enters:

```text
http://ritu.sohandogra.com
```

DNS resolves the hostname to:

```text
3.110.201.212
```

The request is then sent to the EC2 instance.

### Request Flow

```text
Browser
   |
   | http://ritu.sohandogra.com
   v
DNS
   |
   | A Record
   v
3.110.201.212
   |
   v
EC2
```

<!-- Screenshot: Route 53 record showing ritu → 3.110.201.212 -->

---

# 5. Configure Domain on Application

After DNS configuration, NGINX must be configured to recognize the domain name and serve the existing frontend application.

The frontend is already available on the EC2 instance.

The application build is located at:

```text
/home/ubuntu/frontend/build
```

NGINX serves this frontend application.

---

## 5.1 Configure NGINX server_name

Connect to the EC2 instance using SSH.

Open the NGINX configuration:

```bash
sudo nano /etc/nginx/sites-available/ot-poc
```

Configure the server block with the application domain:

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

### Important Configuration

The following directive tells NGINX which domain this server block should handle:

```nginx
server_name ritu.sohandogra.com;
```

The frontend files are served from:

```nginx
root /home/ubuntu/frontend/build;
```

The following configuration supports frontend routes:

```nginx
try_files $uri /index.html;
```

This allows the React application to handle client-side routes correctly.

<!-- Screenshot: NGINX configuration with server_name -->

---

## 5.2 Configure Frontend

The existing frontend application is already hosted on the EC2 instance.

The NGINX configuration points to the frontend build directory:

```text
/home/ubuntu/frontend/build
```

Verify that the frontend build exists:

```bash
ls -la /home/ubuntu/frontend/build
```

Expected output should include:

```text
index.html
```

The domain configured in NGINX is:

```text
ritu.sohandogra.com
```

Therefore, the application request flow becomes:

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

<!-- Screenshot: Frontend build directory / index.html -->

---

## 5.3 Reload NGINX

Before applying the configuration, validate the NGINX configuration:

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

Verify that NGINX is running:

```bash
sudo systemctl status nginx
```

### Expected Result

```text
active (running)
```

<!-- Screenshot: nginx -t successful -->

<!-- Screenshot: NGINX active (running) -->

---

# 6. DNS Validation

After configuring the DNS record and NGINX, verify that the domain resolves correctly and that the application is accessible using the domain.

---

## 6.1 Verify DNS Resolution

Use `nslookup` to verify the DNS resolution:

```bash
nslookup ritu.sohandogra.com
```

The response should contain the EC2 public IP:

```text
3.110.201.212
```

### Expected Result

```text
Name:    ritu.sohandogra.com
Address: 3.110.201.212
```

This confirms that:

```text
ritu.sohandogra.com
        |
        v
3.110.201.212
```

is correctly configured.

<!-- Screenshot: nslookup result -->

---

## 6.2 Access Application Using Domain

Open the following URL in a browser:

```text
http://ritu.sohandogra.com
```

The browser sends the request to the domain.

DNS resolves the domain to the EC2 public IP, and NGINX serves the frontend application.

### Complete Request Flow

```text
User Browser
     |
     | http://ritu.sohandogra.com
     v
DNS Resolution
     |
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

The frontend application opens successfully using:

```text
http://ritu.sohandogra.com
```

The application dashboard and configured frontend pages should be accessible through the domain.

<!-- Screenshot: Application dashboard using ritu.sohandogra.com -->

<!-- Screenshot: Browser address bar showing ritu.sohandogra.com -->

---

# 7. POC Result

The POC was completed successfully.

A custom domain was obtained and configured for the frontend application. The domain `sohandogra.com` is registered through Cloudflare, while AWS Route 53 is used for DNS management.

The subdomain:

```text
ritu.sohandogra.com
```

was configured with an A record pointing to the EC2 public IP:

```text
3.110.201.212
```

NGINX was configured with the same domain using the `server_name` directive and serves the existing frontend application.

The final application flow is:

```text
                    Domain Registration
                         Cloudflare
                             |
                             v
                     sohandogra.com
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
                             |
                             v
                          NGINX
                             |
                             v
                    React Frontend
```

### Final Configuration

| Component | Configuration |
|---|---|
| Registered Domain | `sohandogra.com` |
| Domain Registrar | Cloudflare |
| DNS Management | AWS Route 53 |
| Application Subdomain | `ritu.sohandogra.com` |
| DNS Record | A |
| EC2 Public IP | `3.110.201.212` |
| Web Server | NGINX |
| Frontend | React |
| Application Access | `http://ritu.sohandogra.com` |

---

# 8. Contact Information

| Name | Email Address |
| ---- | ------------- |
| Ritu | ritu.dogra.snaatak@mygurukulam.com |

---

# 9. References

| Reference | Description |
|---|---|
| [AWS Route 53 Documentation](https://docs.aws.amazon.com/route53/) | AWS managed DNS service documentation |
| [Cloudflare Documentation](https://developers.cloudflare.com/) | Domain and DNS documentation |
| [NGINX Documentation](https://nginx.org/en/docs/) | NGINX configuration reference |
| [DNS Basics](https://www.cloudflare.com/learning/dns/what-is-dns/) | Basic explanation of DNS |
| [OT-Microservices Frontend](https://github.com/OT-MICROSERVICES/frontend) | Frontend source repository |

---
