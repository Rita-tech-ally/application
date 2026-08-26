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

To remove the current user's cron jobs:

```bash
crontab -r
```

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
<img width="566" height="69" alt="Screenshot from 2026-08-26 23-15-37" src="https://github.com/user-attachments/assets/e5f1a44c-a840-4c8a-a297-e998addd5a6a" />


### Run every 5 minutes

```bash
*/5 * * * * /path/to/script.sh
```
<img width="566" height="69" alt="Screenshot from 2026-08-26 23-17-49" src="https://github.com/user-attachments/assets/4efdbc9b-8d8f-41c0-b170-bd554f097d0e" />

### Run every 30 sec 

```bash
* * * * * sleep 30; /path/to/script.sh
```
<img width="562" height="89" alt="Screenshot from 2026-08-26 23-26-12" src="https://github.com/user-attachments/assets/3ff79dd8-3526-4c4a-a976-efd4f256b6a5" />

### Run every Monday to Friday at 9:00 AM

```bash
0 9 * * 1-5 /path/to/script.sh
```
<img width="558" height="64" alt="Screenshot from 2026-08-26 23-24-03" src="https://github.com/user-attachments/assets/b3f87140-c478-4285-a2b4-d1e0b90ba781" />

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
nano /home/ritu/backup.sh
```

Example script:

```bash
#!/bin/bash

echo "Backup started at $(date)" >> /home/ritu/backup.log
```

---

## 5.2 Make the Script Executable

Give execute permission:

```bash
chmod +x /home/ritu/backup.sh
```

Verify the permissions:

```bash
ls -l /home/ritu/backup.sh
```
<img width="625" height="49" alt="Screenshot from 2026-08-26 23-55-42" src="https://github.com/user-attachments/assets/20ea2ef9-b7eb-4353-87ef-3e39cececc09" />

---

## 5.3 Configure the Cron Job

Open the user's crontab:

```bash
crontab -e
```

Add the following entry:

```text
0 2 * * * /home/ritu/backup.sh
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
0 2 * * * /home/ritu/backup.sh
```
<img width="564" height="73" alt="Screenshot from 2026-08-26 23-47-13" src="https://github.com/user-attachments/assets/f7b393c5-7125-42c9-8162-e198cf64d550" />

---

# 6. Testing Cron Jobs

It is important to verify the cron configuration and ensure that the scheduled command executes successfully.

## 6.1 Test the Script Manually

Before scheduling the script, execute it manually:

```bash
/home/ritu/backup.sh
```

Check the output log:

```bash
cat /home/ritu/backup.log
```

---

## 6.2 Test with a Frequent Schedule

For testing purposes, configure the cron job to run every minute:

```text
* * * * * /home/ritu/backup.sh
```
<img width="564" height="73" alt="Screenshot from 2026-08-26 23-47-13" src="https://github.com/user-attachments/assets/9e74ce39-150d-4a40-a83a-8553278d2cb9" />

Wait for a minute and check the log:

```bash
cat /home/ritu
/backup.log
```

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
0 1 * * * /home/user/backup.sh
```
<img width="559" height="87" alt="Screenshot from 2026-08-26 23-49-25" src="https://github.com/user-attachments/assets/8e874c2c-3e4b-41f9-94f6-8a4301d59f5d" />


## 7.2 Redirect Output to a Log File

Cron jobs can redirect standard output and errors to a log file:

```text
0 2 * * * /home/user/backup.sh >> /home/user/cron.log 2>&1
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

On systems using systemd journal:

```bash
journalctl -u cron
```

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

| Name | Email Address |
| ---- | ------------- |
| Ritu |               |

---

# 12. References

| Links                                                | Description                                     |
| ---------------------------------------------------- | ----------------------------------------------- |
| https://man7.org/linux/man-pages/man5/crontab.5.html | Linux crontab manual and Cron syntax reference. |
| https://man7.org/linux/man-pages/man8/cron.8.html    | Linux cron service manual.                      |
| https://man7.org/linux/man-pages/man8/crond.8.html   | Linux crond service documentation.              |
