# SOP for Cron Jobs

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| ------ | ---------- | ------- | ----------- | ----------- | ----------- |
| Ritu   | 26-08-2026 | v1.0    |             |             |             |

---

# Table of Contents

1. [Purpose](#1-purpose)
2. [Prerequisites](#2-prerequisites)
3. [Cron Configuration Structure](#3-cron-configuration-structure)
4. [Cron Job Syntax and Key Parameters](#4-cron-job-syntax-and-key-parameters)
5. [Configuration Steps](#5-configuration-steps)
6. [Testing Cron Jobs](#6-testing-cron-jobs)
7. [Automation and Scheduling](#7-automation-and-scheduling)
8. [Monitoring and Troubleshooting](#8-monitoring-and-troubleshooting)
9. [Conclusion](#9-conclusion)
10. [FAQs](#10-faqs)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

# 1. Purpose

Many Linux administration tasks need to be executed automatically at specific times or at regular intervals. Examples include backups, log cleanup, system monitoring, report generation, database maintenance, and script execution.

**Cron** is a time-based job scheduler available in Linux and Unix-like operating systems. It allows administrators and users to schedule commands and scripts to run automatically at predefined times.

Cron can be used to:

* Schedule commands and scripts.
* Run tasks periodically.
* Automate system maintenance.
* Execute backup scripts.
* Perform log cleanup.
* Generate scheduled reports.
* Run monitoring or health-check scripts.
* Automate repetitive administrative tasks.

This SOP explains how to configure and manage **Cron Jobs**, including cron syntax, scheduling, configuration, testing, automation, monitoring, and troubleshooting.

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

> **Note:** `crontab -r` removes all cron jobs for the current user. Use it carefully.

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

### Run every 5 minutes

```bash
*/5 * * * * /path/to/script.sh
```
<img width="861" height="133" alt="Screenshot from 2026-08-27 14-32-58" src="https://github.com/user-attachments/assets/78247c4c-ddbd-4ea1-bb00-b7bbf4147890" />

### Run every 30 sec 

```bash
* * * * * sleep 30; /path/to/script.sh
```
<img width="769" height="154" alt="Screenshot from 2026-08-27 14-34-42" src="https://github.com/user-attachments/assets/51f25ede-e6de-46b1-8535-71bebbf61f96" />


### Run every Monday to Friday at 9:00 AM

```bash
23 13 * * 1-5 /path/to/backup.sh
```
<img width="862" height="153" alt="Screenshot from 2026-08-27 14-25-25" src="https://github.com/user-attachments/assets/7b3d49b4-17ff-4b27-a0ee-593127e14693" />

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
cat /home/ubuntu
/log.file
```
<img width="715" height="241" alt="log" src="https://github.com/user-attachments/assets/f20683e2-aa18-44f6-9b19-f36147ef21d2" />


After successful testing, change the cron schedule to the required production schedule.

---

## 6.3 Verify Cron Service

Ubuntu/Debian:

```bash
systemctl status cron
```

RHEL/CentOS/Amazon Linux:

```bash
systemctl status crond
```

---

# 7. Automation and Scheduling

Cron is designed to execute commands and scripts automatically according to the configured schedule.

Cron can be used for several automation tasks, including:

* Database backups.
* Application maintenance.
* Log cleanup.
* Temporary file cleanup.
* System health checks.
* Report generation.
* Application scripts.
* Data synchronization.
* Monitoring tasks.

## 7.1 Example: Automated Backup

A backup script can be scheduled every day at 1:00 AM:

```text
0 1 * * * /home/ubuntu/backup.sh
```
<img width="559" height="87" alt="Screenshot from 2026-08-26 23-49-25" src="https://github.com/user-attachments/assets/8e874c2c-3e4b-41f9-94f6-8a4301d59f5d" />


## 7.2 Redirect Output to a Log File

Cron jobs can redirect standard output and errors to a log file:

```text
0 2 * * * /home/ubuntu/backup.sh >> /home/user/cron.log 2>&1
```
<img width="556" height="109" alt="Screenshot from 2026-08-26 23-50-01" src="https://github.com/user-attachments/assets/83544106-8372-4dc1-b9ee-94d6a8f0c09d" />

Here:

* `>>` appends standard output to the log file.
* `2>&1` redirects error output to the same log file.

---

# 8. Monitoring and Troubleshooting

## 8.1 Check Configured Cron Jobs

Display the current user's cron jobs:

```bash
crontab -l
```
<img width="556" height="109" alt="Screenshot from 2026-08-26 23-50-01" src="https://github.com/user-attachments/assets/2caffd5b-c763-4635-aaf2-8dcbf356adc1" />

---

## 8.2 Check Cron Logs

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

## 8.3 Common Issues

| Issue                                       | Cause                                                       | Solution                                                                      |
| ------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Cron job is not running                     | Cron service may not be running                             | Check `systemctl status cron` or `systemctl status crond`.                    |
| Incorrect schedule                          | Cron expression is incorrect                                | Verify the five cron fields and test with a frequent schedule.                |
| Script does not execute                     | Script does not have execute permission                     | Use `chmod +x /path/to/script.sh`.                                            |
| Command works manually but not through Cron | Cron has a limited environment/PATH                         | Use absolute paths for commands and files.                                    |
| No output is generated                      | Output is not redirected                                    | Redirect output and errors to a log file.                                     |
| Permission denied                           | User does not have required permissions                     | Verify ownership, permissions, and whether the task requires root privileges. |
| Environment variables are missing           | Cron does not load the normal interactive shell environment | Define required variables in the crontab or use absolute paths.               |
| Job runs at the wrong time                  | Time zone or system time configuration issue                | Verify system date, time, and time zone using `date` and `timedatectl`.       |

---

# 9. Conclusion

Cron provides a simple and reliable mechanism for automating repetitive tasks on Linux systems. It allows administrators and users to schedule commands and scripts at specific times or intervals.

Proper Cron configuration helps automate:

* Backups.
* Log cleanup.
* System maintenance.
* Monitoring.
* Report generation.
* Application-related tasks.

Before deploying a Cron Job in a production environment, always verify the schedule, test the command manually, confirm that the Cron service is running, and configure appropriate logging.

Regular monitoring of Cron jobs helps ensure that scheduled tasks are executed successfully and allows administrators to identify failures quickly.

---

# 10. FAQs

* **What is Cron?**

  * Cron is a time-based job scheduler in Linux used to automatically execute commands and scripts at predefined times.

* **How can I create a Cron Job?**

  * Use the following command:

```bash
crontab -e
```

* Then add the required schedule and command.

* **How can I list Cron Jobs?**

  * Use:

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

* **Can Cron run scripts?**

  * Yes, Cron can execute shell scripts and other executable commands at scheduled times.

* **How can I run a Cron Job every 5 minutes?**

  * Use:

```text
*/5 * * * * /path/to/script.sh
```

* **How can I check whether Cron is running?**

  * On Ubuntu/Debian:

```bash
systemctl status cron
```

* On RHEL/CentOS/Amazon Linux:

```bash
systemctl status crond
```

---

# 11. Contact Information

| Name |         Email Address             |
| ---- | ----------------------------------|
| Ritu | ritu.dogra.snaatak@mygurukulam.co |

---

# 12. References

| Links                                                | Description                                     |
| ---------------------------------------------------- | ----------------------------------------------- |
| https://man7.org/linux/man-pages/man5/crontab.5.html | Linux crontab manual and Cron syntax reference. |
| https://man7.org/linux/man-pages/man8/cron.8.html    | Linux cron service manual.                      |
| https://man7.org/linux/man-pages/man8/crond.8.html   | Linux crond service documentation.              |
