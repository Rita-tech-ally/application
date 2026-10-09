# POC | Domain and DNS Setup for Frontend Application

<p align="center">
  <img width="90" height="auto" alt="dns-icon" src="https://img.icons8.com/fluency/96/domain.png" />
</p>

---

## Author Information

| Author | Created On | Version | Last Updated By | Last Edited On | L0 Reviewer      | L1 Reviewer | L2 Reviewer            |
| ------ | ---------- | ------- | --------------- | -------------- | ---------------- | ----------- | ---------------------- |
| Vikas  | 29-09-2026 | v1.0    | Vikas           | 29-09-2026     | Liyakhat/Anirudh | Aman Raj    | Sandeep Rawat/Ravindra |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Objective](#2-objective)
3. [Architecture](#3-architecture)
4. [Prerequisites](#4-prerequisites)
5. [Infrastructure Configuration](#5-infrastructure-configuration)
6. [Domain Setup](#6-domain-setup)
7. [DNS Configuration](#7-dns-configuration)
8. [NGINX Configuration](#8-nginx-configuration)
9. [Validation](#9-validation)
10. [Result](#10-result)
11. [Conclusion](#11-conclusion)
12. [Contact](#12-contact)
13. [References](#13-references)

---

# 1. Introduction

This document describes the process of registering a custom domain and configuring it for a frontend application that is already hosted on an AWS EC2 instance and served through NGINX.

The domain `devsecurity.shop` was registered through Hostinger, and AWS Route 53 was used as the DNS management service. The Route 53 name servers were configured at the registrar, and an **A record** was created to point the domain to the public IP address of the EC2 instance.

NGINX was then configured with the custom domain as its `server_name` so that it serves the existing frontend build for requests received on that domain. The setup was validated through DNS resolution checks and by accessing the application at `http://devsecurity.shop`.

---

# 2. Objective

The objectives of this POC were to:

* Register a custom domain through Hostinger.
* Create a public hosted zone in AWS Route 53.
* Delegate DNS management to Route 53 by configuring its name servers at the registrar.
* Create an A record mapping the domain to the EC2 public IP address.
* Configure the NGINX `server_name` directive for the custom domain.
* Validate DNS resolution and frontend accessibility through the custom domain.

---

# 3. Architecture

The POC used the following components:

| Component        | Role                                 |
| ---------------- | ------------------------------------ |
| AWS EC2          | Frontend hosting server              |
| NGINX            | Web server                           |
| Frontend build   | Existing static frontend application |
| AWS Route 53     | DNS management and hosted zone       |
| Hostinger        | Domain registration                  |

### Request and DNS Flow

```text
User Browser
     |
     |  http://devsecurity.shop
     v
AWS Route 53  (A record)
     |
     v
15.252.181.35  (EC2 public IP)
     |
     v
NGINX
     |
     v
Frontend Application
```

---

# 4. Prerequisites

The following resources were required for this POC:

| Requirement          | Purpose                        |
| -------------------- | ------------------------------ |
| Hostinger account    | Domain registration            |
| Registered domain    | Application domain             |
| AWS account          | Access to Route 53 and EC2     |
| Route 53 access      | DNS management                 |
| Running EC2 instance | Hosts the frontend             |
| EC2 public IP        | A record target                |
| NGINX                | Serves the frontend            |
| SSH access           | NGINX configuration            |
| Internet access      | DNS and application validation |

---

# 5. Infrastructure Configuration

An existing AWS EC2 instance was used for frontend hosting.

| Parameter           | Configuration                 |
| ------------------- | ----------------------------- |
| Public IP           | `15.252.181.35`               |
| Web Server          | NGINX                         |
| Frontend Build Path | `/home/ubuntu/frontend/build` |

### Security Group

The EC2 security group was configured to allow the following traffic:

| Port | Protocol | Source          | Purpose                                      |
| ---- | -------- | --------------- | -------------------------------------------- |
| 22   | TCP      | Your IP address | SSH access to the EC2 instance               |
| 80   | TCP      | `0.0.0.0/0`     | HTTP traffic to NGINX                        |
| 443  | TCP      | `0.0.0.0/0`     | HTTPS traffic, for SSL/TLS configured later  |

---

# 6. Domain Setup

## 6.1 Register the Domain

A custom domain provides a readable hostname for the frontend application. The domain used in this POC is:

```text
devsecurity.shop
```

The domain was registered through Hostinger as follows:

1. Log in to Hostinger.
2. Open the domain registration section.
3. Search for the required domain.
4. Register `devsecurity.shop`.
5. Verify that the domain status is **Active**.
6. Confirm that the domain is listed in the Hostinger domain management section.

---

## 6.2 Create the Route 53 Hosted Zone

AWS Route 53 was used to manage the DNS records for the domain.

1. Log in to the AWS Management Console.
2. Open **Route 53**.
3. Select **Hosted zones**.
4. Click **Create hosted zone**.
5. Enter the domain name `devsecurity.shop`.
6. Select **Public hosted zone** as the type.
7. Click **Create hosted zone**.

Route 53 automatically creates the hosted zone along with the required NS and SOA records.

---

## 6.3 Configure Name Servers (Delegation)

Because the domain is registered with Hostinger while DNS is managed by Route 53, the Route 53 name servers must be configured at the registrar.

1. Open the Route 53 hosted zone for `devsecurity.shop`.
2. Locate the **NS** record and copy the four name servers.
3. Log in to Hostinger and open the DNS / nameserver settings for `devsecurity.shop`.
4. Replace the existing name servers with the Route 53 name servers.
5. Save the changes.

> **Note:** Nameserver changes can take some time to propagate across the internet.

---

# 7. DNS Configuration

An **A record** maps a domain name to an IPv4 address. The following record was created in the Route 53 hosted zone for `devsecurity.shop`:

| Parameter      | Value           |
| -------------- | --------------- |
| Record Name    | Blank (root)    |
| Record Type    | A               |
| Routing Policy | Simple          |
| Alias          | No              |
| Value          | `15.252.181.35` |
| TTL            | 60 seconds      |

### Steps

1. Open the Route 53 hosted zone for `devsecurity.shop`.
2. Click **Create record**.
3. Leave the record name empty to target the root domain.
4. Select record type **A**.
5. Enter the EC2 public IP address `15.252.181.35` as the value.
6. Set the TTL to `60` seconds.
7. Click **Create records**.

The A record maps the custom domain to the public IP address of the EC2 instance.

---

# 8. NGINX Configuration

After the DNS configuration, NGINX was configured to respond to the custom domain. The frontend application was already hosted on the EC2 instance, and NGINX serves the existing build from:

```text
/home/ubuntu/frontend/build
```

## 8.1 Configure `server_name`

Connect to the EC2 instance and open the NGINX site configuration:

```bash
sudo nano /etc/nginx/sites-available/ot-poc
```

Configure the server block as follows:

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

The `server_name` directive instructs NGINX to handle requests received for `devsecurity.shop`. The `try_files` directive falls back to `index.html`, which allows React client-side routes to be handled correctly.

## 8.2 Verify the Frontend Build

Confirm that the existing frontend build is present:

```bash
ls -la /home/ubuntu/frontend/build
```

The resulting application flow is:

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

## 8.3 Test and Reload NGINX

Validate the configuration syntax:

```bash
sudo nginx -t
```

Reload NGINX and verify its status:

```bash
sudo systemctl reload nginx
sudo systemctl status nginx
```

The configuration test reported no errors, and the NGINX service was active after the reload.

---

# 9. Validation

The setup was validated at the DNS level and at the application level.

## 9.1 DNS Resolution

DNS resolution was verified using:

```bash
nslookup devsecurity.shop
```

**Expected result:** the domain resolves to `15.252.181.35`.

**Observed result:** the domain resolved to the EC2 public IP address as expected.

## 9.2 Application Access

The frontend application was accessed in a browser using:

```text
http://devsecurity.shop
```

The browser resolved the domain through DNS and sent the request to the EC2 instance. NGINX received the request and served the frontend application successfully.

---

# 10. Result

The POC successfully demonstrated:

* Domain registration through Hostinger.
* Creation of a public hosted zone in AWS Route 53.
* Delegation of DNS management by configuring the Route 53 name servers at the registrar.
* An A record mapping the domain to the EC2 public IP address.
* NGINX configuration with the custom domain as `server_name`.
* Successful DNS resolution of `devsecurity.shop`.
* Successful frontend access through `http://devsecurity.shop`.

---

# 11. Conclusion

This POC demonstrated how to register a custom domain and configure it for an existing frontend application hosted on AWS EC2 and served by NGINX.

The domain `devsecurity.shop` was registered through Hostinger, DNS was managed through AWS Route 53, and an A record was created to map the domain to the EC2 public IP. NGINX was configured to serve the application for this domain, and the setup was validated through DNS resolution and browser access.

---

# 12. Contact

| Name           | Email                                                                                   |
| -------------- | --------------------------------------------------------------------------------------- |
| Vikas Badliwal | [vikash.badliwal.snaatak@mygurukulam.co](mailto:vikash.badliwal.snaatak@mygurukulam.co) |

---

# 13. References

| Links                                                                                      | Resource                          |
| ------------------------------------------------------------------------------------------ | --------------------------------- |
| [OT-Microservices Frontend Repository](https://github.com/OT-MICROSERVICES)                | Frontend source code              |
| [AWS Route 53 Documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/)   | DNS and hosted zone configuration |
| [NGINX Documentation](https://nginx.org/en/docs/)                                          | Web server configuration          |
| [Hostinger Domain Help](https://support.hostinger.com/en/collections/1738339-domains)      | Domain and nameserver management  |
