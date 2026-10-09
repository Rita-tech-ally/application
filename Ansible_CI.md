# POC | Domain and DNS Setup for Frontend Application

<p align="center">
  <img width="90" height="auto" alt="dns-icon" src="https://img.icons8.com/fluency/96/domain.png" />
</p>

---

## Author Information

| Author | Created On | Version | Last Updated By | Last Edited On | L0 Reviewer            | L1 Reviewer    | L2 Reviewer        |
| ------ | ---------- | ------- | --------------- | -------------- | ---------------------- | -------------- | ------------------ |
| Vikas  | 16-09-2026 | v1.0    | Vikas           | 16-09-2026     | Deepak Kushwaha/Ayushi | Faisal/Mohit K | Mahesh Kumar/Varun |
| Vikas  | 29-09-2026 | v1.1    | Vikas           | 29-09-2026     | Deepak Kushwaha/Ayushi | Faisal/Mohit K | Mahesh Kumar/Varun |

---
# Table of Contents

1. [Introduction](#1-introduction)
2. [Objective](#2-objective)
3. [Architecture](#3-architecture)
4. [Prerequisites](#4-prerequisites)
5. [Infrastructure Summary](#5-infrastructure-summary)
6. [Domain Setup Summary](#6-domain-setup-summary)
7. [Result](#7-result)
8. [Conclusion](#8-conclusion)
9. [Contact](#9-contact)
10. [References](#10-references)


---

# 1. Introduction

This document provides an overview of the custom domain setup for an existing frontend application hosted on an AWS EC2 instance and served through NGINX.

The domain `devsecurity.shop` was registered through Hostinger, while AWS Route 53 was used to manage DNS records. NGINX was configured to serve the existing frontend application using the custom domain.

For detailed information, refer to the related **[POC | Frontend Hosting with DNS ](https://github.com/SnaatakAllStars/Sprint-1/blob/SCRUM-114-RITU/Documentation/Domain_Security/DNS_SSL/DNS/POC/README.md)**

---

# 2. Objective

The objective of this POC was to connect a custom domain to an existing frontend application.

- Register the domain `devsecurity.shop` through Hostinger.
- Use AWS Route 53 for DNS management.
- Point the domain to the EC2 public IP address.
- Configure NGINX to respond to the custom domain.
- Verify frontend accessibility through the domain.

---

# 3. Architecture

## 3.1 Components and Their Roles

| Component | Role |
|---|---|
| **Hostinger** | Domain registration |
| **AWS Route 53** | Public hosted zone and DNS record management |
| **AWS EC2** | Hosts the existing frontend application |
| **NGINX** | Serves the existing frontend build |
| **Frontend Build** | Contains the static application files |

## 3.2  Request Flow

```text
User's Browser
      |
      v
devsecurity.shop
      |
      v
AWS Route 53
      |
      v
A Record (EC2 Public IP)
      |
      v
AWS EC2 Instance
      |
      v
NGINX Web Server
      |
      v
Existing Frontend Application
```

---

# 4. Prerequisites

| Requirement | Purpose |
|---|---|
| **Hostinger account and registered domain** | Domain registration and management |
| **AWS account with Route 53 access** | DNS management |
| **Existing EC2 instance** | Frontend application hosting |
| **EC2 public IP address** | Target for the DNS A record |
| **SSH access** | Access to the EC2 instance and NGINX configuration |
| **NGINX** | Serves the frontend application |
| **Internet access** | DNS and browser accessibility verification |

**Security Group Requirements:**

- **Port 22 (SSH):** Allow access from the authorized user's IP address.
- **Port 80 (HTTP):** Allow access from the required clients.
- **Port 443 (HTTPS):** Required if HTTPS is configured.

---

# 5. Infrastructure Summary

An existing EC2 instance was used to host the frontend application. Creating the EC2 instance and deploying the frontend application are outside the scope of this document.

| Parameter | Configuration |
|---|---|
| **Domain** | `devsecurity.shop` |
| **EC2 Public IP (documented)** | `15.252.181.35` |
| **Web Server** | NGINX** |
| **Frontend Build Path** | `/home/ubuntu/frontend/build` |
| **HTTP URL** | [http://devsecurity.shop](http://devsecurity.shop) |


---

# 6. Domain Setup Summary

The following configuration was used to connect the custom domain to the existing frontend application:

1. The domain `devsecurity.shop` was registered through Hostinger.
2. A public hosted zone was created in AWS Route 53.
3. The Route 53 name servers were configured in Hostinger to delegate DNS management to Route 53.
4. An A record was configured for the root domain to point to the EC2 public IP address.
5. NGINX was configured with `devsecurity.shop` as its `server_name`.
6. The existing frontend build was served from `/home/ubuntu/frontend/build`.
7. DNS resolution and browser accessibility were validated.

The detailed execution commands, configuration changes, and validation evidence are maintained in the separate POC to avoid duplication.

---

# 7. Result

The domain and frontend integration was documented using the following components:

| Component | Description |
|---|---|
| **Hostinger** | Domain registration |
| **AWS Route 53** | DNS management and domain resolution |
| **AWS EC2** | Hosts the existing frontend application |
| **NGINX** | Serves the frontend application using the custom domain |

The separate POC contains the detailed implementation steps and validation evidence.

---

# 8. Conclusion

This document summarizes the approach used to connect `devsecurity.shop` to an existing frontend application hosted on AWS EC2.

Hostinger was used for domain registration, AWS Route 53 for DNS management, and NGINX for serving the frontend application. The detailed configuration and validation procedures are maintained separately in the POC.

---


# 9. Contact

| Name           | Email                                                                                   |
| -------------- | --------------------------------------------------------------------------------------- |
| Vikas Badliwal | [vikash.badliwal.snaatak@mygurukulam.co](mailto:vikash.badliwal.snaatak@mygurukulam.co) |

---

# 10. References

| Links                                                                                      | Resource                          |
| ------------------------------------------------------------------------------------------ | --------------------------------- |
| [OT-Microservices Frontend Repository](https://github.com/OT-MICROSERVICES)                | Frontend source code              |
| [AWS Route 53 Documentation](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/)   | DNS and hosted zone configuration |
| [NGINX Documentation](https://nginx.org/en/docs/)                                          | Web server configuration          |
| [Hostinger Domain Help](https://support.hostinger.com/en/collections/1738339-domains)      | Domain and nameserver management  |
