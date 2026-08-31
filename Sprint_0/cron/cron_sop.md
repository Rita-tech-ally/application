   <img width="1000" height="420" alt="image" src="https://github.com/user-attachments/assets/e0ca5795-b2e2-40f7-9df0-a52a1a85d9b1" />

# SOP for Cron Jobs
## Document Information


| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 26/08/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |
---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Cron Configuration Structure](#3-cron-configuration-structure)
4. [Cron Job Syntax and Key Parameters](#4-cron-job-syntax-and-key-parameters)
5. [Configuration Steps](#5-configuration-steps)
6. [Testing Cron Jobs](#6-testing-cron-jobs)
7. [Check Cron Logs](#7-check-cron-logs)
8. [Conclusion](#8-conclusion)
9. [FAQs](#9-faqs)
10. [Contact Information](#10-contact-information)
11. [References](#11-references)

# 1. Introduction
Cron is a time-based job scheduler in Unix-like operating systems that automates repetitive tasks and administrative commands at specified intervals.

**Cron:-** A Cron job is a schedule task that allows you to run scripts or commands at a specified intervals. 

Cron can be used to:

Automate system maintenance and backups

Run periodic scripts, logs cleanup, and health checks

Generate scheduled reports

---

# 2. Prerequisites

Cron is generally installed by default on most Linux systems.

Verify that the `crontab` command is available:

```bash
crontab -l
```

You can also check the cron service:

### Ubuntu/Debian

```bash
systemctl status cron
```
<img width="1599" height="518" alt="Screenshot from 2026-08-26 22-47-02" src="https://github.com/user-attachments/assets/ea0105ba-ba74-4004-a6b7-7f209860ecc4" />


### RHEL/CentOS/Amazon Linux

```bash
systemctl status crond
```

If the service is not running, start it using the appropriate command:

```bash
sudo systemctl start cron
```

or:

```bash
sudo systemctl start crond
```

Enable the service to start automatically after system reboot:

```bash
sudo systemctl enable cron
```
<img width="959" height="119" alt="Screenshot from 2026-08-27 14-41-02" src="https://github.com/user-attachments/assets/97069d56-f9f3-40bb-ae6d-f06f7cd249e4" />

or:

```bash
sudo systemctl enable crond
```

---

# 3. Cron Configuration Structure

Cron jobs can be configured for individual users as well as for system-level tasks.

The commonly used cron configuration locations are:

```text
/var/spool/cron/crontabs/
```
<img width="961" height="80" alt="Screenshot from 2026-08-27 14-43-11" src="https://github.com/user-attachments/assets/9c167287-c421-4113-9f2d-f8d21f6a4531" />

and:

```text
/etc/crontab
/etc/cron.d/
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```
---

## 3.1 User Crontab

Each user can maintain their own cron jobs using:

```bash
crontab -e
```
<img width="862" height="247" alt="Screenshot from 2026-08-26 22-59-37" src="https://github.com/user-attachments/assets/8d37b935-e1cf-4ddb-a278-0412066ae16e" />

To display the current user's cron jobs:

```bash
crontab -l
```
<img width="862" height="153" alt="Screenshot from 2026-08-27 14-15-14" src="https://github.com/user-attachments/assets/fa6a9e35-a043-4071-b7e4-8061855dd230" />

To remove the current user's cron jobs:

```bash
crontab -r
```
<img width="1311" height="314" alt="Screenshot from 2026-08-27 13-41-06" src="https://github.com/user-attachments/assets/6c83520e-1938-45a5-ba2c-2ead8efcbe50" />


---

## 3.2 System Cron Configuration

The main system-wide cron configuration file is:

```bash
/etc/crontab
```

System-specific cron jobs can also be maintained under:

```bash
/etc/cron.d/
```

Periodic jobs can be placed in:

```text
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

---

# 4. Cron Job Syntax and Key Parameters

A standard user cron entry contains five scheduling fields followed by the command:

```text
* * * * * command
```

The five fields are:

```text
┌───────────── minute (0 - 59)
│ ┌─────────── hour (0 - 23)
│ │ ┌───────── day of month (1 - 31)
│ │ │ ┌─────── month (1 - 12)
│ │ │ │ ┌───── day of week (0 - 7)
│ │ │ │ │
* * * * * command
```

Both `0` and `7` generally represent Sunday.

----

## 4.1 Common Cron Scheduling Parameters

| Parameter    | Description                     | Example     |
| ------------ | ------------------------------- | ----------- |
| `*`          | Represents every possible value | `* * * * *` |
| `,`          | Specifies multiple values       | `1,15`      |
| `-`          | Specifies a range of values     | `1-5`       |
| `/`          | Specifies an interval           | `*/5`       |
| Minute       | Specifies minute of execution   | `30`        |
| Hour         | Specifies hour of execution     | `2`         |
| Day of Month | Specifies day of the month      | `15`        |
| Month        | Specifies month                 | `8`         |
| Day of Week  | Specifies day of the week       | `1-5`       |

## 4.2 Common Examples

### Run every minute

```bash
* * * * * /path/to/script.sh
```

<img width="865" height="88" alt="Screenshot from 2026-08-27 14-28-06" src="https://github.com/user-attachments/assets/453ffbe1-b2db-451b-8ffb-d49685d768c3" />

----
### Run every 5 minutes

```bash
*/5 * * * * /path/to/script.sh
```
<img width="861" height="133" alt="Screenshot from 2026-08-27 14-32-58" src="https://github.com/user-attachments/assets/78247c4c-ddbd-4ea1-bb00-b7bbf4147890" />

----

### Run the script 30 seconds after every minute 

```bash
* * * * * sleep 30; /path/to/script.sh
```
<img width="769" height="154" alt="Screenshot from 2026-08-27 14-34-42" src="https://github.com/user-attachments/assets/51f25ede-e6de-46b1-8535-71bebbf61f96" />

Note: Cron does not have a seconds field. The sleep 30 command delays execution by 30 seconds after Cron starts the
command each minute. Therefore, this runs the script at approximately 30 seconds past each minute, not every 30 seconds.

----

### Run every Monday to Friday at 1:23 PM

```bash
23 13 * * 1-5 /path/to/backup.sh
```
<img width="862" height="153" alt="Screenshot from 2026-08-27 14-25-25" src="https://github.com/user-attachments/assets/7b3d49b4-17ff-4b27-a0ee-593127e14693" />

----

# 5. Configuration Steps

| Step | Action                      | Command / Configuration          | Description                                                     |
| ---- | --------------------------- | -------------------------------- | --------------------------------------------------------------- |
| 1    | Create or identify a script | `/home/user/backup.sh`           | Identify the command or script that needs to run automatically. |
| 2    | Make the script executable  | `chmod +x /home/user/backup.sh`  | Provides execute permission to the script.                      |
| 3    | Open crontab                | `crontab -e`                     | Opens the user's cron configuration.                            |
| 4    | Add schedule                | `0 2 * * * /home/user/backup.sh` | Runs the script every day at 2:00 AM.                           |
| 5    | Verify configuration        | `crontab -l`                     | Displays configured cron jobs.                                  |

---

## 5.1 Create a Script

First create the script that needs to be executed.

Example:

```bash
nano /home/ubuntu/backup.sh
```

Example script:

```bash
#!/bin/bash

echo "Backup started at $(date)" >> /home/ubuntu/backup.log
```

---

## 5.2 Make the Script Executable

Give execute permission:

```bash
chmod +x /home/ubuntu/backup.sh
```

Verify the permissions:

```bash
ls -l /home/ubuntu/backup.sh
```
<img width="730" height="102" alt="Screenshot from 2026-08-27 12-54-25" src="https://github.com/user-attachments/assets/e861f5fb-65f0-4f34-8fc4-6ea141f9e460" />


---

## 5.3 Configure the Cron Job

Open the user's crontab:

```bash
crontab -e
```

Add the following entry:

```text
0 2 * * * /home/ubuntu/backup.sh
```

This configuration runs the script **every day at 2:00 AM**.

---

## 5.4 Verify the Cron Job

To verify that the cron job has been configured:

```bash
crontab -l
```

Expected output:

```text
0 2 * * * /home/ubuntu/backup.sh
```
<img width="710" height="152" alt="Screenshot from 2026-08-27 12-56-32" src="https://github.com/user-attachments/assets/b8fac8b3-8b1e-4bf8-8297-269efce9321b" />

---

# 6. Testing Cron Jobs

It is important to verify the cron configuration and ensure that the scheduled command executes successfully.

## 6.1 Test the Script Manually

Before scheduling the script, execute it manually:

```bash
/home/ubuntu/backup.sh
```

Check the output log:

```bash
cat /home/ubuntu/log.file
```

---

## 6.2 Test with a Frequent Schedule

For testing purposes, configure the cron job to run every minute:

```text
* * * * * /home/ubuntu/script.sh
```

<img width="710" height="152" alt="Screenshot from 2026-08-27 12-58-01" src="https://github.com/user-attachments/assets/23b1b437-90a2-4807-863f-ed88c6b1dd49" />


Wait for a minute and check the log:

```bash
cat /home/ubuntu/log.file
```
<img width="715" height="241" alt="log" src="https://github.com/user-attachments/assets/f20683e2-aa18-44f6-9b19-f36147ef21d2" />


After successful testing, change the cron schedule to the required production schedule.

---

## 7 Check Cron Logs

On Ubuntu/Debian systems, cron activity can commonly be checked using:

```bash
grep CRON /var/log/syslog
```
<img width="1316" height="69" alt="Screenshot from 2026-08-27 13-34-14" src="https://github.com/user-attachments/assets/4e6766d5-4d1e-40cf-ae85-34ef0c09c111" />

On systems using systemd journal:

```bash
journalctl -u cron
```
<img width="1316" height="69" alt="Screenshot from 2026-08-27 13-33-19" src="https://github.com/user-attachments/assets/18ea5c69-f036-4576-a77a-5f14fe3e5702" />

For RHEL/CentOS/Amazon Linux:

```bash
journalctl -u crond
```

---

# 8. Conclusion

Cron is a reliable Linux utility that automates repetitive tasks like backups, log cleanup, and system maintenance by scheduling commands at specific intervals. Before deploying jobs to production, always test commands, verify schedules, and set up proper logging. Regular monitoring is essential to ensure successful execution and quickly catch any failures.

---

# 9. FAQs

* **What is Cron?**

  * Cron is a time-based job scheduler in Linux used to automatically execute commands and scripts at predefined times.

* **How can I create a Cron Job?**

  * Use the following command: 

```bash
crontab -l
```

* **How can I remove Cron Jobs?**

  * Use:

```bash
crontab -r
```

* This removes all Cron Jobs for the current user, so it should be used carefully.

* **What are the five fields in Cron syntax?**

  * The five fields are:

```text
Minute
Hour
Day of Month
Month
Day of Week
```

# 10. Contact Information

| Name |         Email Address             |
| ---- | ----------------------------------|
| Ritu | ritu.dogra.snaatak@mygurukulam.co |

---

# 11. References

| Links                                                | Description                                     |
| ---------------------------------------------------- | ----------------------------------------------- |
| https://man7.org/linux/man-pages/man5/crontab.5.html | Linux crontab manual and Cron syntax reference. |
| https://man7.org/linux/man-pages/man8/cron.8.html    | Linux cron service manual.                      |
| https://man7.org/linux/man-pages/man8/crond.8.html   | Linux crond service documentation.              |
