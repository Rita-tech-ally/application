# DNS POC | Domain Setup for Application

<p align="center">
  <img width="90" height="auto" alt="dns-icon" src="https://img.icons8.com/fluency/96/domain.png" />
</p>

---

# Author Table

| **Author**    | **Created On** | **Version** | **Last Edited On** | **L0 Reviewer** | **L1 Reviewer** | **L2 Reviewer** |
| :------------ | :------------- | :---------- | :----------------- | :-------------- | :-------------- | :-------------- |
| Anshul Kotiya | 14-09-2026     | 1.0         | 14-09-2026         | Shubham Rathi   | Shreya / Nikita | Piyush Upadhyay |

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Application Setup](#3-application-setup)

   * [3.1 Create EC2 Instance](#31-create-ec2-instance)
   * [3.2 Configure Security Group](#32-configure-security-group)
   * [3.3 Connect to EC2](#33-connect-to-ec2)
   * [3.4 Install NGINX](#34-install-nginx)
   * [3.5 Create Application](#35-create-application)
   * [3.6 Configure NGINX](#36-configure-nginx)
   * [3.7 Validate Application](#37-validate-application)
4. [Domain Setup](#4-domain-setup)

   * [4.1 Create Domain](#41-create-domain)
   * [4.2 Configure Domain with EC2 Public IP](#42-configure-domain-with-ec2-public-ip)
5. [DNS Validation](#5-dns-validation)

   * [5.1 Verify DNS Resolution](#51-verify-dns-resolution)
   * [5.2 Access Application Using Domain](#52-access-application-using-domain)
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
| Instance Name     | `dns-poc-server` |
| Instance Type     | `t3.micro`       |
| Region            | `eu-north-1`     |
| Operating System  | Ubuntu           |
| Public IPv4       | `13.48.193.66`   |

After the instance is running, note the **Public IPv4 address**.

<img width="1920" height="1140" alt="Screenshot 2026-09-14 144949" src="https://github.com/user-attachments/assets/7632e9c9-5ebb-4557-adf2-436108820d36" />


---

## 3.2 Configure Security Group

Configure the EC2 Security Group to allow the required traffic.

| **Type** | **Port** | **Source**      |
| -------- | -------: | --------------- |
| SSH      |       22 | Your IP address |
| HTTP     |       80 | `0.0.0.0/0`     |
| HTTPS    |      443 | `0.0.0.0/0`     |


---

## 3.3 Connect to EC2

Connect to the EC2 instance using SSH:

```bash
ssh -i <KEY_FILE>.pem ubuntu@13.48.193.66
```

Verify the connection:

```bash
hostname
```

**Expected Result:**

The hostname of the EC2 instance should be displayed.


---

## 3.4 Install NGINX

Update the package repository:

```bash
sudo apt update
```

Install NGINX:

```bash
sudo apt install -y nginx
```

Enable and start NGINX:

```bash
sudo systemctl enable --now nginx
```

Validate the NGINX configuration:

```bash
sudo nginx -t
```

**Expected Result:**

```text
syntax is ok
test is successful
```

Check the NGINX service:

```bash
sudo systemctl status nginx
```

**Expected Result:**

The NGINX service should show:

```text
active (running)
```

<img width="1026" height="762" alt="Screenshot 2026-09-14 145342" src="https://github.com/user-attachments/assets/a740d3d8-cb40-4e0c-bcd1-b7957b80ff49" />


---

## 3.5 Create Application

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

## 3.6 Configure NGINX

Create an NGINX configuration file:

```bash
sudo nano /etc/nginx/sites-available/dns-poc
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name mywebsite-dns.duckdns.org;

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

<img width="1920" height="1140" alt="Screenshot 2026-09-14 150749" src="https://github.com/user-attachments/assets/59121d56-fa50-4108-b988-752181e72ab8" />


---

# 4. Domain Setup

## 4.1 Create Domain

For this POC, a free domain is created using **DuckDNS**.

Sign in to DuckDNS and create the following hostname:

```text
mywebsite-dns.duckdns.org
```

Verify that the hostname is displayed in the DuckDNS dashboard.


---

## 4.2 Configure Domain with EC2 Public IP

In the DuckDNS dashboard, configure the created domain with the public IP address of the EC2 instance.

Set the IP address to:

```text
13.48.193.66
```

The final mapping should be:

```text
mywebsite-dns.duckdns.org → 13.48.193.66
```


<img width="1920" height="1140" alt="Screenshot 2026-09-14 160516" src="https://github.com/user-attachments/assets/4e53b94e-313e-48a4-8f47-152a53f2e54c" />


---

# 5. DNS Validation

## 5.1 Verify DNS Resolution

Open Command Prompt or Terminal and run:

```bash
nslookup mywebsite-dns.duckdns.org
```

**Expected Result:**

```text
Name:    mywebsite-dns.duckdns.org
Address: 13.48.193.66
```

The domain should resolve to the public IP address of the EC2 instance.

<img width="787" height="168" alt="Screenshot 2026-09-14 151729" src="https://github.com/user-attachments/assets/ba26cd20-ed86-468c-85ad-30452afb3d5a" />


---

## 5.2 Access Application Using Domain

Open the following URL in a browser:

```text
http://mywebsite-dns.duckdns.org
```

**Expected Result:**

The application hosted on the EC2 instance should be accessible using the configured domain.

<img width="1920" height="1140" alt="Screenshot 2026-09-14 153336" src="https://github.com/user-attachments/assets/e6f6c12c-73c6-43b7-a4a2-f9d31f2068a9" />


---

# 6. POC Result

The DNS POC was successfully completed.

The domain:

```text
mywebsite-dns.duckdns.org
```

was configured to point to the AWS EC2 public IP:

```text
13.48.193.66
```

The application was successfully accessed using the configured domain.

---

# 7. Contact Information

| **Name**      | **Email**                                                                           |
| ------------- | ----------------------------------------------------------------------------------- |
| Anshul Kotiya | [anshul.kotiya.snaatak@mygurukulam.co](mailto:anshul.kotiya.snaatak@mygurukulam.co) |

---

# 8. References

| **Reference**                                                                                                                                                                  | **Description**                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------- |
| [DNS Documentation — Sprint-1 | SnaatakKubeClan](https://github.com/SnaatakKubeClan/Sprint-1/blob/SCRUM-110-PRIYANSHU/Documentation/Domain_Security/DNS_SSL/DNS/DOC/README.md) | Project DNS documentation reference.                     |


---
