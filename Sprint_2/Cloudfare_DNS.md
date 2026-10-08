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
