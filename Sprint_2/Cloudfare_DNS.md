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

For this POC, the domain `sohandogra.com` is registered through Cloudflare, while AWS Route 53 is used for DNS management.

The frontend application is already hosted on an AWS EC2 instance using NGINX. The purpose of this POC is to configure the subdomain `ritu.sohandogra.com`, map it to the EC2 public IP using an A record, and configure NGINX to serve the frontend using the domain name.

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

The following requirements are needed for this POC:

| Requirement | Purpose |
|---|---|
| Cloudflare account | Domain registration |
| Registered domain | Base domain |
| AWS account | Route 53 and EC2 access |
| Route 53 access | DNS management |
| Running EC2 instance | Hosts the frontend |
| EC2 Public IP | Target of the A record |
| NGINX | Serves the frontend |
| SSH access | Configure NGINX |
| Internet access | DNS and application validation |

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

The application uses the following subdomain:

```text
ritu.sohandogra.com
```

Here:

```text
sohandogra.com
```

is the registered domain and:

```text
ritu
```

is the subdomain label.

Therefore:

```text
ritu.sohandogra.com
```

is the complete hostname used to access the application.

### Steps

1. Log in to the Cloudflare account.
2. Search for the required domain.
3. Register the domain if it is not already registered.
4. Verify that the domain is active.
5. Use the registered domain for DNS configuration.

<!-- Screenshot: Cloudflare domain registration / domain overview -->

---

## 3.2 Create Route 53 Hosted Zone

After obtaining the domain, create a public hosted zone in AWS Route 53.

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

Route 53 creates the hosted zone and automatically provides NS and SOA records.

The NS record contains the name servers that are used for DNS delegation.

<!-- Screenshot: Route 53 hosted zone -->

<!-- Screenshot: Route 53 NS records -->

---

## 3.3 Configure Name Servers / Delegation

The domain is registered through Cloudflare, while DNS management for this POC is performed through AWS Route 53.

Therefore, the Route 53 name servers need to be configured for the domain's DNS delegation.

Route 53 provides four name servers similar to:

```text
ns-xxx.awsdns-xx.org
ns-xxx.awsdns-xx.com
ns-xxx.awsdns-xx.net
ns-xxx.awsdns-xx.co.uk
```

### Steps

1. Open the Route 53 hosted zone.
2. Locate the **NS record**.
3. Copy the Route 53 name servers.
4. Open the domain DNS/name-server settings in Cloudflare.
5. Configure the required Route 53 name-server delegation.
6. Save the configuration.

After delegation, DNS queries for the configured domain can be handled through Route 53.

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
    |
    v
DNS Records
```

<!-- Screenshot: Route 53 NS records -->

<!-- Screenshot: Cloudflare nameserver configuration -->

---

# 4. DNS Configuration

## 4.1 Create Subdomain

The application is exposed through the following subdomain:

```text
ritu.sohandogra.com
```

The subdomain label is:

```text
ritu
```

and the registered domain is:

```text
sohandogra.com
```

The complete hostname is therefore:

```text
ritu.sohandogra.com
```

This hostname will be mapped to the public IP address of the EC2 instance.

### DNS Structure

```text
sohandogra.com
      |
      +---- ritu.sohandogra.com
```

<!-- Screenshot: Route 53 hosted zone -->

---

## 4.2 Create A Record

An A record maps a hostname to an IPv4 address.

For this POC, an A record is created for:

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
5. Enter the EC2 public IP as the value.
6. Set TTL to `60` seconds.
7. Keep the routing policy as `Simple`.
8. Click **Create records**.

<!-- Screenshot: Route 53 Create A Record -->

---

## 4.3 Point A Record to EC2 Public IP

The A record points the application subdomain to the EC2 public IPv4 address.

For this POC:

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

When a user enters:

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
   | ritu.sohandogra.com
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

<!-- Screenshot: Route 53 A record showing ritu → 3.110.201.212 -->

---

# 5. Configure Domain on Application

Once DNS is configured, the application web server must be configured to respond to the domain.

The frontend is already hosted on the EC2 instance.

NGINX is used as the web server and serves the existing frontend build.

The purpose of this step is to tell NGINX that:

```text
ritu.sohandogra.com
```

should be served by the frontend application.

---

## 5.1 Configure NGINX server_name

Connect to the EC2 instance using SSH and open the NGINX configuration:

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

The important configuration is:

```nginx
server_name ritu.sohandogra.com;
```

This tells NGINX to handle requests made for the configured domain.

The frontend build is served from:

```nginx
root /home/ubuntu/frontend/build;
```

The `try_files` configuration allows the React frontend to handle client-side routes.

<!-- Screenshot: NGINX configuration -->

---

## 5.2 Configure Frontend

The frontend application is already available on the EC2 instance.

The NGINX configuration points to the existing frontend build:

```text
/home/ubuntu/frontend/build
```

Verify that the build directory contains the application files:

```bash
ls -la /home/ubuntu/frontend/build
```

The directory should contain:

```text
index.html
```

The application request flow is:

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

Before applying the configuration, test the NGINX configuration:

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

Verify the NGINX service:

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

After configuring the DNS record and NGINX, validate that the domain resolves to the correct EC2 public IP and that the application is accessible through the domain.

---

## 6.1 Verify DNS Resolution

Use `nslookup` to verify the DNS resolution:

```bash
nslookup ritu.sohandogra.com
```

The response should contain:

```text
3.110.201.212
```

### Expected Result

```text
Name:    ritu.sohandogra.com
Address: 3.110.201.212
```

This confirms that the DNS A record is resolving the subdomain to the EC2 public IP.

### DNS Mapping

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

The browser first resolves the domain through DNS.

After DNS resolution, the request reaches the EC2 instance, where NGINX serves the frontend application.

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

<!-- Screenshot: Application dashboard using domain -->

<!-- Screenshot: Browser address bar showing ritu.sohandogra.com -->

---

# 7. POC Result

The POC was completed successfully.

The domain `sohandogra.com` was obtained through Cloudflare and DNS management was configured using AWS Route 53.

A Route 53 hosted zone was created and the required DNS delegation was configured.

The application subdomain:

```text
ritu.sohandogra.com
```

was configured with an A record pointing to:

```text
3.110.201.212
```

the public IP address of the EC2 instance.

NGINX was configured with the same hostname using the `server_name` directive and serves the existing frontend application.

The domain was successfully validated using `nslookup`, and the frontend application was accessed through:

```text
http://ritu.sohandogra.com
```

### Final Architecture

```text
                    Cloudflare
                 Domain Registration
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
                    ot-poc
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
| Application | React Frontend |
| Application URL | `http://ritu.sohandogra.com` |

---

# 8. Contact Information

| Name | Email Address |
|---|---|
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
