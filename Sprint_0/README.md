<p align="center">
<img width="440" height="120" alt="image" src="https://github.com/user-attachments/assets/f46115f8-d4bc-4357-8355-6b78ee8d9cf3" />
</p>

---

# Ansible Role CD Workflow | Documentation

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer            |
| ------ | ---------- | ------- | ----------- | ----------- | ---------------------- |
| Ritu   | 09/09/2026 | 1.0     | Liyakhat    | Aman Raj    | Sandeep Rawat/Ravindra |

---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Prerequisites](#2-prerequisites)
3. [Ansible Role](#3-ansible-role)
4. [What is Continuous Deployment (CD)?](#4-what-is-continuous-deployment-cd)
5. [Why Use Ansible Role in CD?](#5-why-use-ansible-role-in-cd)
6. [CD Workflow](#6-cd-workflow)
7. [Jenkins Pipeline](#7-jenkins-pipeline)
8. [Deployment Process](#8-deployment-process)
9. [Conclusion](#9-conclusion)
10. [Contact Information](#10-contact-information)
11. [Reference](#11-reference)

---

# 1. Purpose

This document describes the Continuous Deployment (CD) workflow for an Ansible Role.

The workflow automates the deployment of validated Ansible code to target servers after the required CI checks have successfully passed.

The main goal is to reduce manual deployment effort and ensure consistent and reliable application of Ansible Roles across target environments.

---

# 2. Prerequisites

The following components are required:

| Component             | Purpose                                         |
| --------------------- | ----------------------------------------------- |
| **Git**               | Stores Ansible Role and configuration code      |
| **GitHub**            | Manages repository, branches, and Pull Requests |
| **Jenkins**           | Automates the CD pipeline                       |
| **Ansible**           | Automates configuration and deployment          |
| **Ansible Role**      | Contains reusable automation tasks              |
| **Ansible Inventory** | Defines target servers                          |
| **SSH Access**        | Provides connectivity to target servers         |
| **CI Pipeline**       | Validates Ansible code before deployment        |

---

# 3. Ansible Role

An **Ansible Role** is a reusable structure used to organize automation code.

The role contains tasks, handlers, templates, files, variables, defaults, and other components required for configuration and deployment.

Typical role structure:

<img width="214" height="418" alt="image" src="https://github.com/user-attachments/assets/557ca0be-4032-4baa-97c6-597393d3371c" />

---

# 4. What is Continuous Deployment (CD)?

Continuous Deployment (CD) is a DevOps practice in which validated code changes are automatically deployed to the target environment without requiring manual deployment steps.

In an Ansible-based CD workflow, Jenkins triggers Ansible automation to apply the required configuration or deployment changes on target servers.

---

# 5. Why Use Ansible Role in CD?

Ansible Roles are useful in CD because they provide a structured and reusable way to automate deployments and server configuration.

| **Benefit**               | **Description**                                                               |
| ------------------------- | ----------------------------------------------------------------------------- |
| **Automated Deployment**  | Automatically applies configuration and deployment changes to target servers. |
| **Reusable Automation**   | Ansible Roles can be reused across multiple environments and servers.         |
| **Consistent Deployment** | Applies the same deployment process across target servers.                    |
| **Idempotency**           | Repeated execution produces the desired state without unnecessary changes.    |
| **Agentless**             | No Ansible agent is required on managed servers.                              |
| **Scalability**           | Can deploy changes to multiple servers simultaneously.                        |
| **Jenkins Integration**   | Jenkins can automatically trigger Ansible deployment jobs.                    |
| **Reduced Manual Effort** | Minimizes manual configuration and deployment activities.                     |

---

# 6. CD Workflow

The CD workflow automatically deploys validated Ansible changes to the target environment.

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/f4187f14-e565-4050-858a-081f4ac98665" />

---

### Workflow Steps

| Step   | Activity                                        |
| ------ | ----------------------------------------------- |
| **1**  | Developer completes Ansible code changes        |
| **2**  | Code is pushed to Git                           |
| **3**  | Pull Request is created or updated              |
| **4**  | CI pipeline validates the Ansible code          |
| **5**  | Required CI checks pass successfully            |
| **6**  | Code is merged into the deployment branch       |
| **7**  | Jenkins CD pipeline is triggered                |
| **8**  | Jenkins checks out the latest approved code     |
| **9**  | Required Ansible dependencies are installed     |
| **10** | Ansible inventory and configuration are loaded  |
| **11** | Ansible Role is executed against target servers |
| **12** | Deployment/configuration changes are applied    |
| **13** | Deployment result is verified                   |
| **14** | Jenkins generates the deployment result         |
| **15** | Deployment is marked as successful or failed    |

---

# 7. Jenkins Pipeline

Jenkins automates the Continuous Deployment process.

### Pipeline Stages

| Stage                    | Purpose                                             |
| ------------------------ | --------------------------------------------------- |
| **Checkout**             | Gets the latest approved Ansible code               |
| **Setup**                | Installs required Ansible dependencies              |
| **Inventory**            | Loads the target server inventory                   |
| **Pre-Deployment Check** | Performs required checks before deployment          |
| **Deploy**               | Executes the Ansible Role on target servers         |
| **Verification**         | Verifies that the deployment completed successfully |
| **Report**               | Reports the deployment result                       |

---

# 8. Deployment Process

The deployment process follows these steps:

### Step 1: Checkout

Jenkins checks out the approved Ansible Role and required configuration from the Git repository.

### Step 2: Setup

Jenkins prepares the environment and installs the required Ansible dependencies and collections.

### Step 3: Inventory

The Ansible inventory identifies the servers where the Role needs to be applied.

### Step 4: Pre-Deployment Check

Required validation and connectivity checks are performed before applying changes to the target servers.

### Step 5: Ansible Role Execution

Jenkins executes the Ansible Playbook that uses the required Role.

Ansible connects to the target servers through SSH and applies the required configuration or deployment changes.

### Step 6: Verification

After deployment, required checks are performed to verify that the changes were successfully applied.

### Step 7: Deployment Result

Jenkins reports the final deployment status as **Success** or **Failure**.

---

# 9. Conclusion

The Ansible CD workflow automates the deployment of validated Ansible Roles to target servers.

By integrating Ansible with Jenkins, the deployment process becomes consistent, repeatable, and less dependent on manual activities.

This helps ensure that approved changes are deployed reliably across the required environments.

---

# 10. Contact Information

| Name | Email Address                                                                 |
| ---- | ----------------------------------------------------------------------------- |
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

---

# 11. Reference

| Link                       | Description                    |
| -------------------------- | ------------------------------ |
| https://docs.ansible.com   | Official Ansible documentation |
| https://www.jenkins.io/doc | Jenkins documentation          |
