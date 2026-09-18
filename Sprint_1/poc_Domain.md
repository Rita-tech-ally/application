# DNS POC | Domain Setup for Application

<p align="center">
  <img width="90" height="auto" alt="dns-icon" src="https://img.icons8.com/fluency/96/domain.png" />
</p>

# Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Application Setup](#3-application-setup)
4. [Domain Setup](#4-domain-setup)
5. [DNS Validation](#5-dns-validation)
6. [POC Result](#6-poc-result)
7. [Contact Information](#7-contact-information)
8. [References](#8-references)

---

# 1. Introduction

This document demonstrates the DNS POC by obtaining a domain and configuring it to access an application hosted on an AWS EC2 instance.



---

# 2. Prerequisites

The following are required to perform this POC:

* AWS account
* AWS EC2 instance
* SSH private key
* Internet access
* DuckDNS account
* NGINX

---

# 3. Application Setup

## 3.1 Create EC2 Instance

Create an Ubuntu EC2 instance in AWS.

The following EC2 instance was used for this POC:

| **Configuration** | **Value**        |
| ----------------- | ---------------- |
| Instance Name     | `DNS_poc-server` |
| Instance Type     | `t3.micro`       |
| Region            | `ap-south-1b`     |
| Operating System  | Ubuntu           |
| Public IPv4       | `15.207.254.122`   |

After the instance is running, note the **Public IPv4 address**.

<img width="1348" height="667" alt="Screenshot from 2026-09-18 09-37-02" src="https://github.com/user-attachments/assets/72dbd950-5878-477e-9e8d-8181567e9220" />



---

## 3.2 Configure Security Group

Configure the EC2 Security Group to allow the required traffic.

| **Type** | **Port** | **Source**      |
| -------- | -------: | --------------- |
| SSH      |       22 | Your IP address |
| HTTP     |       80 | `0.0.0.0/0`     |
| HTTPS    |      443 | `0.0.0.0/0`     |


---


## 3.3 Install NGINX

Install NGINX:

```bash
sudo apt install -y nginx
```

<img width="839" height="195" alt="Screenshot from 2026-09-18 09-32-59" src="https://github.com/user-attachments/assets/ee44b2ff-203b-4543-ba34-adccdd389c00" />

Enable and start NGINX:

```bash
sudo systemctl enable nginx
```
<img width="1377" height="84" alt="Screenshot from 2026-09-18 09-33-53" src="https://github.com/user-attachments/assets/eaa10a11-2916-4196-a3de-4af5f85e7c93" />

Validate the NGINX configuration:

```bash
sudo nginx -t
```

**Expected Result:**

```text
syntax is ok
test is successful
```

<img width="839" height="85" alt="Screenshot from 2026-09-18 09-31-00" src="https://github.com/user-attachments/assets/9ee2309d-f6f3-4417-84f8-c66d24c6e71d" />


Check the NGINX service:

```bash
sudo systemctl status nginx
```

**Expected Result:**

The NGINX service should show:

```text
active (running)
```

<img width="1027" height="353" alt="Screenshot from 2026-09-18 09-29-55" src="https://github.com/user-attachments/assets/3c036b00-7ae0-4865-bf4a-7c7a9920bf68" />



---

## 3.4 Create Application

Create the application directory:

```bash
sudo mkdir -p /var/www/dns-poc
```

Create the application file:

```bash
sudo nano /var/www/dns-poc/index.html
```

Add the following HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to My Website</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding-top: 150px;
            background: #F4F4F4;
        }

        .container {
            background: white;
            max-width: 600px;
            margin: auto;
            padding: 50px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }

        h1 {
            font-size: 40px;
        }

        p {
            font-size: 18px;
        }
    </style>
</head>

<body>

    <div class="container">
        <h1>Welcome to My Website</h1>

        <p>Hello and welcome!</p>

        <p>
            This website is hosted on AWS EC2 using NGINX.
        </p>

        <p>
            <strong>DNS POC</strong>
        </p>
    </div>

</body>
</html>
```

Save the file and exit.

---

## 3.5 Configure NGINX

Create an NGINX configuration file:

```bash
sudo nano /etc/nginx/sites-available/dns-poc
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name devsecurity.shop;

    root /var/www/dns-poc;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/dns-poc /etc/nginx/sites-enabled/dns-poc
```

Remove the default NGINX configuration:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Validate the configuration:

```bash
sudo nginx -t
```

Reload NGINX:

```bash
sudo systemctl reload nginx
```

---

## 3.7 Validate Application

Verify that the application is available locally:

```bash
curl http://localhost
```

Open the application using the EC2 public IP:

```text
http://13.48.193.66
```

**Expected Result:**

The application page should be displayed successfully.

<img width="1316" height="538" alt="Screenshot from 2026-09-18 10-01-57" src="https://github.com/user-attachments/assets/4f01f14c-eb67-4731-95a6-c04605a46575" />

# 4. Domain Setup

## 4.1 Create Domain

For this POC, a free domain is created using **Hostinger**.

Sign in to Hostinger and create the following hostname:

```text
devsecurity.shop
```

Verify that the hostname is displayed in the Hostinger dashboard.


---



# 5. DNS Validation

## 5.1 Verify DNS Resolution

Open Command Prompt or Terminal and run:

```bash
nslookup devsecurity.shop
```

**Expected Result:**

```text
Name:    devsecurity.shop
Address: 15.207.254.122
```

The domain should resolve to the public IP address of the EC2 instance.

<img width="536" height="159" alt="Screenshot from 2026-09-18 11-07-09" src="https://github.com/user-attachments/assets/a8aa623b-3bef-4a15-8578-6d1978ba395b" />

---

## 5.2 Access Application Using Domain

Open the following URL in a browser:

```text
http://devsecurity.shop
```

**Expected Result:**

The application hosted on the EC2 instance should be accessible using the configured domain.

<img width="1318" height="580" alt="image" src="https://github.com/user-attachments/assets/bf95041f-89f9-4d13-9e71-7c9bc245ab38" />


---

# 6. POC Result

The DNS POC was successfully completed.

The domain:

```text
devsecurity.shop
```

was configured to point to the AWS EC2 public IP:

```text
15.207.254.122
```

The application was successfully accessed using the configured domain.

---
# 7. Contact Information

| Name |         Email Address             |
|-------|-----------------------------------
| Ritu | ritu.dogra.snaatak@mygurukulam.co— |

----

## 8. References

| **Reference**                                                                                                                                                                  | **Description**                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| [DNS Documentation — Sprint-1  | [AllStar] (https://github.com/SnaatakAllStars/Sprint-1/edit/SCRUM-113-MEHAK/Documentation/Domain_Security/DNS_SSL/DNS/DOC/README.md)| Project DNS documentation reference.                                                               |

