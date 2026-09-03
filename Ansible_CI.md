<img width="220" height="120" alt="image" src="https://github.com/user-attachments/assets/f46115f8-d4bc-4357-8355-6b78ee8d9cf3" />

---

# Ansible Role CI Workflow | Documentation

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 03/09/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |


## Table of Contents

1. [Purpose](#1-purpose)
2. [Prerequisites](#2-prerequisites)
3. [Ansible Role](#3-ansible-role)
4. [CI Workflow](#4-ci-workflow)
5. [Jenkins Pipeline](#5-jenkins-pipeline)
6. [CI Validation Checks](#6-ci-validation-checks)
7. [Conclusion](#7-conclusion)
8. [FAQs](#8-faqs)
9. [Contact Information](#7-contact-information)
10. [Reference](#8-reference)


---

# 1. Purpose

This document describes the Continuous Integration (CI) workflow for an Ansible Role.
The workflow automatically checks Ansible code whenever a change is pushed or a Pull Request is created.
The main goal is to identify syntax, linting, and configuration issues before the code is merged.

---

# 2. Prerequisites

The following are required:

| Component         | Purpose                              |
| ----------------- | ------------------------------------ |
| **Git**               | Stores Ansible code                  |
| **GitHub**            | Manages repository and Pull Requests |
| **Jenkins**           | Runs the CI pipeline                 |
| **Ansible**           | Validates Ansible code               |
| **ansible-lint**      | Checks Ansible coding standards      |
| **Ansible Inventory** | Defines target servers                |
| **SSH Access**        | Required for connectivity testing    |

---

# 3. Ansible Role

An **Ansible Role** is a reusable structure used to organize Ansible automation.

Typical role structure:

<img width="214" height="418" alt="image" src="https://github.com/user-attachments/assets/557ca0be-4032-4baa-97c6-597393d3371c" />

### Main Role Directories

| Directory    | Purpose                    |
| ------------ | -------------------------- |
| **`tasks/`**     | Contains deployment tasks  |
| **`handlers/`**  | Contains handlers          |
| **`templates/`** | Contains Jinja2 templates  |
| **`files/`**     | Contains static files      |
| **`vars/`**      | Contains role variables    |
| **`defaults/`**  | Contains default variables |
| **`meta/`**      | Contains role metadata     |

---


# 4. CI Workflow

The CI workflow automatically validates Ansible changes before they are merged into the main branch.

<img width="563" height="1024" alt="image" src="https://github.com/user-attachments/assets/488501c6-c509-415c-98f9-87c87ca38e8f" />


----

### Workflow Steps

| Step | Activity                                      |
| ---- | --------------------------------------------- |
| **1**    | Developer creates/updates Ansible code        |
| **2**    | Code is pushed to Git                         |
| **3**    | Pull Request is created                       |
| **4**    | Jenkins starts the CI pipeline                |
| **5**    | Jenkins checks out the latest code            |
| **6**    | Ansible syntax check is performed             |
| **7**    | Ansible linting is performed                  |
| **8**    | Ansible dry-run/check mode is performed       |
| **9**    | Required tests are executed                   |
| **10**   | CI pipeline result is generated               |
| **11**   | Pull Request is updated with the CI result    |
| **12**   | Code can be merged if all required checks pass|

---

# 5. Jenkins Pipeline

Jenkins automates the CI validation process.

### Pipeline Stages

| Stage        | Purpose                                      |
| ------------ | -------------------------------------------- |
| **Checkout**     | Gets the latest Ansible code                 |
| **Setup**        | Installs required Ansible dependencies      |
| **Syntax Check** | Checks for Ansible syntax errors             |
| **Lint**         | Checks Ansible coding standards              |
| **Dry Run**      | Tests what changes Ansible would make        |
| **Test**         | Executes required automated tests            |
| **Report**       | Reports the CI pipeline result               |

---

# 6. CI Validation Checks

The CI pipeline performs the following checks before code is merged:

| Check                  | Command                                            | Purpose                                      |
| ---------------------- | -------------------------------------------------- | -------------------------------------------- |
| **Syntax Check**       | `ansible-playbook --syntax-check deploy.yml`       | Checks Ansible syntax errors                 |
| **Lint Check**         | `ansible-lint .`                                   | Checks coding standards and best practices   |
| **Dry Run**            | `ansible-playbook -i inventory deploy.yml --check` | Shows expected changes without applying them |
| **Connectivity Check** | `ansible all -i inventory -m ping`                 | Verifies connection with target servers      |

# 7. Conclusion

The Ansible CI workflow automatically validates code using syntax checks, linting, dry runs, and connectivity checks. It helps
identify issues early and ensures that only validated code moves forward for deployment.

----
# 8. FAQs
 ### Q1. What is Continuous Integration (CI)?

  Continuous Integration is a practice where code changes are automatically built, tested, and validated before they are merged.

### Q2. What is the purpose of CI for an Ansible Role?

  CI validates the Ansible Role and identifies syntax, linting, and configuration issues before deployment.

### Q3. What does Jenkins do in the CI workflow?

  Jenkins automatically executes the CI pipeline and performs the required validation checks. 

  ----
# 9. Contact Information

| Name |         Email Address             |
|-------|-----------------------------------
| Ritu | ritu.dogra.snaatak@mygurukulam.co— |

----

# 10. Reference

| Topic                     | Reference                                                                            |
| ------------------------- | ------------------------------------------------------------------------------------ |
| **Ansible Documentation** | Official documentation for Ansible concepts, playbooks, roles, modules, and commands |
| **Ansible Roles**         | Official guide for creating and using reusable Ansible Roles                         |
| **Ansible Lint**          | Documentation for Ansible code quality and best-practice checks                      |
| **Jenkins Documentation** | Official documentation for Jenkins pipelines and CI automation                       |

