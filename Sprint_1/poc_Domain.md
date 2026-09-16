<p align="center">

# POC: Get a Domain and Set It Up for an Application

</p>

---

## Document Information

| Author | Created On | Version | Last Edited On | L0 Reviewer | L1 Reviewer | L2 Reviewer            |
| ------ | ---------- | ------- | -------------- | ----------- | ----------- | ---------------------- |
| Ritu   | 16-09-2026 | v1.0    | 16-09-2026     | Liyakhat    | Aman Raj    | Sandeep Rawat/Ravindra |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Objective](#2-objective)
3. [POC Overview](#3-poc-overview)
4. [Architecture](#4-architecture)
5. [Prerequisites](#5-prerequisites)
6. [Domain Details](#6-domain-details)
7. [Application Setup](#7-application-setup)
8. [Create Subdomain](#8-create-subdomain)
9. [Configure DNS](#9-configure-dns)
10. [Verify DNS Resolution](#10-verify-dns-resolution)
11. [Configure Nginx](#11-configure-nginx)
12. [Configure HTTPS](#12-configure-https)
13. [Verify Application Using Domain](#13-verify-application-using-domain)
14. [Final Request Flow](#14-final-request-flow)
15. [Troubleshooting](#15-troubleshooting)
16. [POC Result](#16-poc-result)
17. [Key Learnings](#17-key-learnings)
18. [Conclusion](#18-conclusion)

---

# 1. Introduction

A domain name provides a human-readable way to access an application instead of using an IP address.

For example, without a domain, an application may be accessed using:

```text
http://<EC2-PUBLIC-IP>:8080
```

After configuring a domain, the same application can be accessed using:

```text
https://ritu.sohandogra.com
```

In this POC, an existing domain is used to create a subdomain and connect it to an application running on an AWS EC2 instance.

The domain used for this POC is:

```text
sohandogra.com
```

The application subdomain is:

```text
ritu.sohandogra.com
```

---

# 2. Objective

The objective of this POC is to demonstrate the complete process of setting up a domain for an application.

The POC covers:

* Using an existing domain.
* Creating a subdomain.
* Configuring DNS.
* Deploying an application on AWS EC2.
* Connecting the subdomain to the EC2 server.
* Configuring Nginx as a reverse proxy.
* Configuring HTTPS.
* Verifying domain resolution.
* Accessing the application using the domain.

---

# 3. POC Overview

The POC follows the below flow:

```text
Existing Domain
      |
      v
Create Subdomain
      |
      v
Configure DNS
      |
      v
Point DNS to AWS
      |
      v
Deploy Application
      |
      v
Configure Nginx
      |
      v
Configure HTTPS
      |
      v
Access Application
```

Final URL:

```text
https://ritu.sohandogra.com
```

---

# 4. Architecture

## 4.1 POC Architecture

```text
                         User
                           |
                           |
               https://ritu.sohandogra.com
                           |
                           v
                    Hostinger DNS
                           |
                           |
                       A Record
                           |
                           v
                    AWS EC2 Instance
                           |
                           v
                        Nginx
                           |
                     Reverse Proxy
                           |
                           v
                  Application :8080
```

## 4.2 Request Flow

```text
Browser
   |
   | HTTPS Request
   v
ritu.sohandogra.com
   |
   | DNS Lookup
   v
Hostinger DNS
   |
   | Returns EC2 Public IP
   v
AWS EC2
   |
   | Port 443
   v
Nginx
   |
   | Proxy Request
   v
Application :8080
```

---

# 5. Prerequisites

The following are required:

* Hostinger account.
* Existing domain `sohandogra.com`.
* Access to Hostinger DNS management.
* AWS account.
* Ubuntu EC2 instance.
* EC2 public IP.
* SSH access to EC2.
* Application running on EC2.
* Application port.

Example:

```text
Domain:
sohandogra.com

Subdomain:
ritu.sohandogra.com

Application Port:
8080
```

---

# 6. Domain Details

## 6.1 Existing Domain

An existing domain is used for this POC:

```text
sohandogra.com
```

A new domain is not purchased because an existing domain is already available.

---

## 6.2 Subdomain

A subdomain is created for the application:

```text
ritu.sohandogra.com
```

Domain structure:

```text
                 sohandogra.com
                       |
                       |
                 ritu.sohandogra.com
```

Here:

* `sohandogra.com` → existing domain.
* `ritu` → subdomain.
* `ritu.sohandogra.com` → complete hostname.

---

# 7. Application Setup

The application is deployed on an AWS EC2 instance.

For this POC, assume:

```text
EC2:
Ubuntu

Application Port:
8080

EC2 Public IP:
<EC2-PUBLIC-IP>
```

Replace `<EC2-PUBLIC-IP>` with the actual public IP of your EC2 instance.

---

## 7.1 Connect to EC2

Connect to the EC2 instance using SSH:

```bash
ssh -i <key.pem> ubuntu@<EC2-PUBLIC-IP>
```

---

## 7.2 Verify Application

Check whether the application is running:

```bash
curl http://localhost:8080
```

If the application is working, it should return the application's response.

For example:

```text
Application is running
```

---

## 7.3 Verify Application Port

Run:

```bash
ss -lntp
```

Check that the application is listening on port `8080`.

Example:

```text
LISTEN 0 128 0.0.0.0:8080
```

---

# 8. Create Subdomain

The subdomain is created through the DNS configuration of the existing domain.

Open Hostinger:

```text
Hostinger
   ↓
Domains
   ↓
Domain Portfolio
   ↓
sohandogra.com
   ↓
Manage
   ↓
DNS / Nameservers
```

Open the DNS management section.

---

# 9. Configure DNS

## 9.1 Create A Record

An A record maps a hostname to an IPv4 address.

Create the following record:

| Type | Name   | Points To         |
| ---- | ------ | ----------------- |
| A    | `ritu` | `<EC2-PUBLIC-IP>` |

Example:

```text
Type:
A

Name:
ritu

Points to:
13.xx.xx.xx
```

The resulting mapping is:

```text
ritu.sohandogra.com
        |
        v
13.xx.xx.xx
        |
        v
AWS EC2
```

### Important

In the Hostinger DNS name field, use:

```text
ritu
```

not:

```text
ritu.sohandogra.com
```

The complete hostname will become:

```text
ritu.sohandogra.com
```

---

# 10. Verify DNS Resolution

After adding the DNS record, verify that the domain resolves to the EC2 IP.

---

## 10.1 Using dig

Run:

```bash
dig ritu.sohandogra.com
```

Look for the `ANSWER SECTION`.

Expected result:

```text
;; ANSWER SECTION:

ritu.sohandogra.com.    IN    A    <EC2-PUBLIC-IP>
```

---

## 10.2 Using nslookup

Run:

```bash
nslookup ritu.sohandogra.com
```

Expected result:

```text
Name:    ritu.sohandogra.com
Address: <EC2-PUBLIC-IP>
```

This confirms that the subdomain is resolving to the EC2 instance.

---

## 10.3 Verify Using ping

You can also check the DNS resolution using:

```bash
ping ritu.sohandogra.com
```

For DNS verification, the important part is that the hostname resolves to the expected IP. ICMP may be blocked by AWS Security Groups, so a failed ping does not necessarily mean DNS is incorrect.

---

# 11. Configure Nginx

Directly exposing an application on port `8080` is not ideal for a user-facing domain.

Nginx can be used as a reverse proxy.

The architecture becomes:

```text
Internet
   |
   | Port 80 / 443
   v
Nginx
   |
   | Port 8080
   v
Application
```

---

## 11.1 Install Nginx

On Ubuntu EC2:

```bash
sudo apt update
sudo apt install nginx -y
```

Check the Nginx status:

```bash
sudo systemctl status nginx
```

If required:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## 11.2 Create Nginx Configuration

Create a configuration file:

```bash
sudo nano /etc/nginx/sites-available/ritu.sohandogra.com
```

Add:

```nginx
server {
    listen 80;
    server_name ritu.sohandogra.com;

    location / {
        proxy_pass http://127.0.0.1:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Save the file.

---

## 11.3 Enable Nginx Configuration

Create a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/ritu.sohandogra.com \
/etc/nginx/sites-enabled/
```

Test the Nginx configuration:

```bash
sudo nginx -t
```

Expected result:

```text
syntax is ok
test is successful
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

---

# 12. Configure AWS Security Group

The EC2 Security Group should allow the required traffic.

For the Nginx-based setup:

| Protocol | Port | Source    |
| -------- | ---- | --------- |
| TCP      | 22   | Your IP   |
| TCP      | 80   | 0.0.0.0/0 |
| TCP      | 443  | 0.0.0.0/0 |

Port `8080` does not need to be publicly exposed when the application is accessed through Nginx.

The desired flow is:

```text
Internet
   |
   | 80 / 443
   v
Nginx
   |
   | 8080
   v
Application
```

The application can listen on localhost:

```text
127.0.0.1:8080
```

if the application supports this configuration.

---

# 13. Test HTTP Domain

Before configuring HTTPS, test HTTP.

Open:

```text
http://ritu.sohandogra.com
```

The request flow is:

```text
Browser
   |
   v
ritu.sohandogra.com
   |
   v
EC2
   |
   v
Nginx :80
   |
   v
Application :8080
```

If the application response is displayed, the domain-to-application configuration is working.

---

# 14. Configure HTTPS

HTTPS protects communication between the browser and the server.

For this POC, **Let's Encrypt** can be used to obtain a free SSL/TLS certificate.

---

## 14.1 Install Certbot

Install Certbot and the Nginx plugin:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx -y
```

---

## 14.2 Request SSL Certificate

Run:

```bash
sudo certbot --nginx -d ritu.sohandogra.com
```

Certbot will:

1. Validate the domain.
2. Obtain the SSL/TLS certificate.
3. Configure Nginx.
4. Configure HTTPS.
5. Optionally redirect HTTP traffic to HTTPS.

Follow the prompts displayed by Certbot.

---

## 14.3 Verify Certificate

Run:

```bash
sudo certbot certificates
```

The certificate should show:

```text
Certificate Name:
ritu.sohandogra.com
```

---

# 15. Verify HTTPS

Open:

```text
https://ritu.sohandogra.com
```

The application should now be accessible securely.

The final flow is:

```text
Browser
   |
   | HTTPS :443
   v
ritu.sohandogra.com
   |
   v
AWS EC2
   |
   v
Nginx
   |
   | HTTP :8080
   v
Application
```

---

# 16. Verify Using curl

Run:

```bash
curl -I https://ritu.sohandogra.com
```

A successful response may look like:

```text
HTTP/2 200
```

The exact response depends on the application.

---

# 17. Verify DNS Again

Run:

```bash
dig ritu.sohandogra.com
```

The domain should resolve to the expected EC2 public IP.

Example:

```text
ritu.sohandogra.com.    IN    A    <EC2-PUBLIC-IP>
```

---

# 18. Verify Nginx

Check Nginx:

```bash
sudo systemctl status nginx
```

Check configuration:

```bash
sudo nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

---

# 19. Verify Application

Check the application locally:

```bash
curl http://localhost:8080
```

Then check through Nginx:

```bash
curl http://localhost
```

Finally check through the domain:

```bash
curl https://ritu.sohandogra.com
```

This verifies the complete chain:

```text
Application
     ↓
Nginx
     ↓
Domain
     ↓
HTTPS
     ↓
User
```

---

# 20. Final Request Flow

The complete POC flow is:

```text
                         Internet User
                              |
                              |
                              v
                  https://ritu.sohandogra.com
                              |
                              v
                       Hostinger DNS
                              |
                         A Record
                              |
                              v
                    AWS EC2 Public IP
                              |
                              v
                         Nginx :443
                              |
                       Reverse Proxy
                              |
                              v
                       Application :8080
```

---

# 21. Troubleshooting

## 21.1 DNS Not Resolving

Check:

```bash
dig ritu.sohandogra.com
```

Verify:

* A record exists.
* Record name is correct.
* EC2 IP is correct.
* DNS is managed by the expected provider.
* DNS changes have propagated.

---

## 21.2 Application Not Working

Check:

```bash
curl http://localhost:8080
```

If this fails, troubleshoot the application first.

Check the application port:

```bash
ss -lntp
```

---

## 21.3 Nginx Not Working

Check:

```bash
sudo systemctl status nginx
```

Test configuration:

```bash
sudo nginx -t
```

Check logs:

```bash
sudo tail -f /var/log/nginx/error.log
```

---

## 21.4 Domain Works but HTTPS Does Not

Check:

```bash
sudo certbot certificates
```

Also verify:

* Port `443` is allowed.
* Certificate is valid.
* Nginx HTTPS configuration exists.
* Domain resolves correctly.
* DNS is pointing to the correct server.

---

## 21.5 Application Works on EC2 IP but Not Through Domain

Check the request flow:

```text
Domain
   ↓
DNS
   ↓
EC2
   ↓
Nginx
   ↓
Application
```

Verify each layer separately:

```bash
dig ritu.sohandogra.com
```

```bash
curl http://localhost:8080
```

```bash
curl http://localhost
```

```bash
curl https://ritu.sohandogra.com
```

---

# 22. POC Evidence

The following screenshots should be attached to demonstrate the POC.

### Screenshot 1 — Hostinger Domain

Show:

```text
sohandogra.com
```

in the Hostinger Domain Portfolio.

### Screenshot 2 — DNS Configuration

Show the A record:

```text
Type: A
Name: ritu
Points to: <EC2-PUBLIC-IP>
```

### Screenshot 3 — AWS EC2

Show the running EC2 instance and its public IP.

### Screenshot 4 — Application

Show the application running on:

```text
http://<EC2-PUBLIC-IP>:8080
```

### Screenshot 5 — DNS Verification

Show:

```bash
dig ritu.sohandogra.com
```

### Screenshot 6 — Nginx

Show:

```bash
sudo nginx -t
```

with successful output.

### Screenshot 7 — HTTP Domain

Show:

```text
http://ritu.sohandogra.com
```

opening the application.

### Screenshot 8 — SSL Certificate

Show the successful Certbot/SSL configuration.

### Screenshot 9 — Final HTTPS Application

Show:

```text
https://ritu.sohandogra.com
```

opening the application successfully.

---

# 23. POC Result

The POC successfully demonstrates how an existing domain can be connected to an application.

The existing domain:

```text
sohandogra.com
```

was used to configure the application subdomain:

```text
ritu.sohandogra.com
```

The subdomain was configured using an A record and pointed to the AWS EC2 instance.

Nginx was configured as a reverse proxy to forward incoming requests to the application running on port `8080`.

HTTPS was then configured using an SSL/TLS certificate.

The final application URL is:

```text
https://ritu.sohandogra.com
```

---

# 24. Key Learnings

Through this POC, the following concepts were implemented:

* Domain and subdomain.
* Domain DNS management.
* A Record configuration.
* DNS resolution.
* AWS EC2 application deployment.
* Nginx reverse proxy.
* HTTP and HTTPS.
* SSL/TLS certificate.
* Domain-based application access.
* DNS troubleshooting.
* Application connectivity verification.

---

# 25. Conclusion

This POC demonstrates the complete process of setting up a domain for an application.

The final architecture is:

```text
Existing Domain
sohandogra.com
       |
       v
Subdomain
ritu.sohandogra.com
       |
       v
Hostinger DNS
       |
       v
AWS EC2
       |
       v
Nginx
       |
       v
Application
       |
       v
HTTPS
```

The application is finally accessible using:

```text
https://ritu.sohandogra.com
```

This demonstrates how DNS, AWS infrastructure, Nginx, and HTTPS work together to make an application available through a custom domain.

---
