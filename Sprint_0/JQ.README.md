
<img width="447" height="107" alt="image" src="https://github.com/user-attachments/assets/142575b9-e6a0-4bb6-8040-48c7be5aa348" />

----
  
  <h1 align="center" style="font-size:100px;">
  SOP: jq Installation Guide
</h1>

            
# Author Table

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 31/08/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |
---

# Table of Contents

# Table of Contents

1. [Purpose](#1-purpose)
2. [Prerequisites](#2-prerequisites)
3. [Step-by-Step Installation](#3-step-by-step-installation)
4. [Conclusion](#4-conclusion)
5. [FAQ](#5-faq)
6. [References](#6-references)
7. [Contact Information](#7-contact-information)
---

# 1. Purpose

This SOP provides a step-by-step procedure for installing **jq** on a Linux system. It also covers installation verification and checking the location of the installed jq executable.

---

# 2. Prerequisites

Before starting the installation, make sure the following requirements are available:

* Linux system with terminal access
* Internet connectivity
* Sudo privileges
* `apt` package manager

---

# 3. Step-by-Step Installation

## Step 1: Update Package Repository

Update the package information before installing jq.

```bash
sudo apt update
```
<img width="957" height="58" alt="Screenshot from 2026-09-01 19-36-55" src="https://github.com/user-attachments/assets/8036ced4-5f64-4f11-b2b1-1296b59d365d" />

---

## Step 2: Install jq

Install jq using the `apt` package manager.

```bash
sudo apt install jq -y
```
<img width="957" height="169" alt="Screenshot from 2026-09-01 19-38-03" src="https://github.com/user-attachments/assets/f1afbc10-8c5f-43c0-af4b-b5cc506d0c87" />

---

## Step 3: Verify jq Installation

Check whether jq has been installed successfully.

```bash
jq --version
```
<img width="957" height="52" alt="Screenshot from 2026-09-01 19-38-54" src="https://github.com/user-attachments/assets/4f058bc9-f81c-45b1-8242-769ee80ef7b4" />

Expected output will display the installed jq version, for example:

```text
jq-1.7
```

---

## Step 4: Check jq Installation Path

Use the following command to identify the location of the jq executable.

```bash
which jq
```

A successful installation normally returns a path similar to:

```text
/usr/bin/jq
```
<img width="957" height="60" alt="Screenshot from 2026-09-01 19-39-27" src="https://github.com/user-attachments/assets/6aa3ae77-195d-4b2b-bbd3-a50a94f97029" />

---

# 4. Conclusion

jq can be installed easily on Linux using the system package manager. After installation, `jq --version` and `which jq` can be used to verify the installation and locate the executable.

---
# 5. FAQ

### 1. What is jq?

`jq` is a lightweight command-line tool used to **parse, filter, and process JSON data**.

### 2. Why is jq used?

jq is commonly used to extract specific information from JSON output, especially while working with **APIs, AWS CLI, and automation scripts**.

### 3. How do I install jq on Ubuntu?

Use the following commands:

```bash
sudo apt update
sudo apt install jq -y
```

### 4. How can I check whether jq is installed?

Run:

```bash
jq --version
```

If jq is installed, it will display the installed version.

### 6. How can I find the location of jq?

Run:

```bash
which jq
```

Usually, the output is:

```text
/usr/bin/jq
```

---

# 6. References

| Resource                  | Link                         |
| ------------------------- | ---------------------------- |
| jq Official Documentation | https://jqlang.github.io/jq/ |
| Ubuntu Packages           | https://packages.ubuntu.com/ |

---

# 7. Contact Information

| Name  | Email ID                                                                        |
| ----- | ------------------------------------------------------------------------------- |
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

---
