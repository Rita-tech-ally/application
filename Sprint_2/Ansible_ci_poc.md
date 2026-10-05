<p align="center">
<img width="440" height="120" alt="image" src="https://github.com/user-attachments/assets/f46115f8-d4bc-4357-8355-6b78ee8d9cf3" />
</p>  

---

# Ansible Role CI POC

---
## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 03/09/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |

---


## Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Ansible Role Galaxy](#3-ansible-role-galaxy)
4. [CI Workflow](#4-ci-workflow)
5. [CI Environment Setup](#5-ci-environment-setup)
6. [CI Implementation](#7-ci-implementation)
7. [Result](#8-result)
8. [Conclusion](#9-conclusion)
9. [Contact Information](#10-contact-information)
10. [Reference](#11-reference)
---

## 1. Introduction

This POC demonstrates how to perform CI checks for an Ansible Role on a local machine.

The POC validates the Ansible Role using YAML validation, Ansible syntax check, `ansible-lint`, connectivity check, and Ansible check mode before the Role is used for deployment.

---

## 2. Prerequisites

The following tools are required:

| Component | Purpose |
|---|---|
| **Ansible** | Executes and validates Ansible code |
| **ansible-lint** | Checks Ansible coding standards |
| **yamllint** | Validates YAML files |
| **SSH** | Provides connectivity to the target server |
| **Ansible Inventory** | Defines the target server |

---

## 3. Ansible Role galaxy

The Ansible Role used for the POC follows the standard structure:

<p align="center">
<img width="1356" height="388" alt="Screenshot from 2026-10-05 11-59-28" src="https://github.com/user-attachments/assets/868e3a17-80bf-470d-98c6-791f430ccf4b" />

</p>
---

## 4. CI Workflow

The CI checks are performed locally on the Ansible Role.

<p align="center">
<img width="1220" height="120" alt="image" src="https://github.com/user-attachments/assets/fcace10e-3578-4640-abd9-c2ba93912a16" />
</p>

---

## 5. CI Environment Setup

The CI environment is configured with the required tools to validate the Ansible Role.

```bash
ansible --version
ansible-lint --version
```
<img width="1048" height="168" alt="image" src="https://github.com/user-attachments/assets/52239167-6d2f-40f9-8df4-84dbc696a337" />
<img width="890" height="61" alt="image" src="https://github.com/user-attachments/assets/6d193f62-9ac7-4cbc-8729-fae95eb99e95" />

---


## 7. CI Implementation


### Step 1: Run ansible-lint

Run:

```bash
ansible-lint
```

This checks the Ansible Role for coding issues and recommended Ansible practices.

<img width="1347" height="107" alt="image" src="https://github.com/user-attachments/assets/fa85467c-bdae-4dc9-8fe9-192e376d968e" />

---

### Step 2. Syntax Check

Validate the Ansible Playbook syntax to identify any syntax errors before deployment.

```bash
ansible-playbook playbook.yml --syntax-check
```
<img width="1341" height="175" alt="image" src="https://github.com/user-attachments/assets/737ca752-7802-48bf-9068-c65c18a1585b" />

### Step 3. Check Mode (Dry Run)

Run the Ansible Playbook in check mode to preview the changes without applying them to the target server.

```bash
ansible-playbook playbook.yml -i inventory.ini --check
```
<img width="1339" height="495" alt="image" src="https://github.com/user-attachments/assets/06640144-380f-443f-a5f5-6e9b46859ee8" />

### Step 4.CI Validation Result

The Ansible Role successfully passed the configured CI validation checks.

- Ansible Lint
- Playbook Syntax Check
- Ansible Check Mode

These checks confirm that the role is ready for the deployment stage.

---

## 8. Result

The Ansible Role was validated locally using YAML validation, Ansible syntax checking, `ansible-lint`, target connectivity testing, and Ansible check mode.

---

## 9. Conclusion

This POC demonstrates local CI validation for an Ansible Role without using Jenkins. The checks help identify YAML, syntax, linting, connectivity, and configuration issues before the Role is used for deployment.

---

# 10. Contact Information

| Name |         Email Address             |
|-------|-----------------------------------
| Ritu | ritu.dogra.snaatak@mygurukulam.co— |

---

## 11. Reference

| Link | Description |
|------|-------------|
| [Ansible Documentation](https://docs.ansible.com/) | Official Ansible documentation |
| [Ansible Lint Documentation](https://docs.ansible.com/projects/lint/) | Official documentation for Ansible Lint |
| [Ansible Lint - GitHub](https://github.com/ansible/ansible-lint) | Ansible Lint source repository |
| [Ansible Community Tools](https://docs.ansible.com/projects/ansible/latest/community/other_tools_and_programs.html) | Ansible tools including Ansible Lint and yamllint |
