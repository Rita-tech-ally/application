# DevOps Repository Setup

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Repository Structure](#2-repository-structure)
3. [Repository Responsibilities](#3-repository-responsibilities)
4. [Setup Completed](#4-setup-completed)
5. [Current Status](#5-current-status)
6. [Verification](#6-verification)
7. [Contact](#7-contact)
8. [References](#8-references)

---

# 1. Introduction

This repository is created to establish the DevOps repository structure for the OT-Microservices project.

As part of the repository setup, separate repositories are being created for Infrastructure, Ansible, CI/CD, Shared Libraries, and Documentation. This follows the Microrepo strategy, where each DevOps responsibility is maintained independently.

The purpose of this setup is to provide a clear and organized structure for future DevOps implementation activities.

The repositories are maintained under the [**All-starsP19 GitHub Organization**](https://github.com/orgs/All-starsP19/repositories).

---

# 2. Repository Structure

```text
OT-Microservices
│
├── Infrastructure_Repository
│   └── Infrastructure Management
│
├── Ansible_Repository
│   └── Configuration & Deployment Automation
│
├── CI-CD_Repository
│   └── CI/CD Automation
│
├── Shared-Library_Repository
│   └── Reusable CI/CD Components
│
└── Documentation_Repository
    └── DevOps Documentation
```

---

# 3. Repository Responsibilities

## 3.1 Infrastructure_Repository

This repository is dedicated to managing infrastructure-related resources for the OT-Microservices project.

It provides a separate space for infrastructure configuration and Infrastructure as Code (IaC). Keeping infrastructure resources in a dedicated repository helps maintain clear ownership, organized management, and separation from application and other DevOps resources.

<img width="1519" height="89" alt="Screenshot from 2026-10-07 00-44-38" src="https://github.com/user-attachments/assets/92c81318-9caa-4664-af6d-7c0dc774edce" />


---

## 3.2 Ansible_Repository

This repository is dedicated to managing configuration management and deployment automation for the OT-Microservices project.

It provides a separate space for Ansible-related resources, helping maintain server configurations, deployment automation, and other configuration management activities in an organized manner.

<img width="1519" height="89" alt="Screenshot from 2026-10-07 00-45-05" src="https://github.com/user-attachments/assets/1cdd05cb-4bd7-4108-865f-4e03a30bc457" />

---

## 3.3 CI-CD_Repository

This repository is dedicated to managing CI/CD pipeline configuration and automation for the OT-Microservices project.

It provides a separate space for pipeline-related resources, helping organize build, testing, and deployment automation independently from application and infrastructure repositories.

<img width="1491" height="88" alt="image" src="https://github.com/user-attachments/assets/c8e922a9-495b-46a2-acb1-268a6bc7230d" />

---

## 3.4 Shared-Library_Repository

This repository is dedicated to managing reusable CI/CD components and common automation logic for the OT-Microservices project.

It provides a separate space for shared pipeline functions, scripts, and common automation components that can be reused across different CI/CD pipelines.

<img width="1491" height="88" alt="Screenshot from 2026-10-07 00-45-48" src="https://github.com/user-attachments/assets/7d200fdf-a5a3-48e2-babf-ca0b9badef80" />


---

## 3.5 Documentation_Repository

This repository is dedicated to maintaining DevOps documentation and operational references for the OT-Microservices project.

It provides a separate space for maintaining architecture documents, SOPs, deployment guides, operational procedures, and troubleshooting information in an organized manner.

<img width="1463" height="92" alt="Screenshot from 2026-10-07 00-46-19" src="https://github.com/user-attachments/assets/1918d923-4ecc-4f0b-a5c7-b77ecebef14a" />

---

# 4. Setup Completed

The following activities have been completed:

- Identified the required DevOps repositories.
- Created the required repositories.
- Applied descriptive repository names.
- Configured repository visibility.
- Verified repository availability.
- Established the repository structure for future implementation.

---

# 5. Current Status

| Activity | Status |
|---|---|
| Repository planning | Completed |
| Repository creation | Completed |
| Repository naming | Completed |
| Repository visibility configuration | Completed |
| Repository verification | Completed |

---

# 6. Verification

The repository setup can be verified from the project's GitHub organization.

The verification confirms that the required repositories have been created and are available for subsequent implementation work.

### Evidence

<img width="803" height="422" alt="Screenshot from 2026-10-07 00-26-57" src="https://github.com/user-attachments/assets/a804f5e6-59cf-46c6-a008-cfb2738ccc24" />

---

# 7. Contact

| Name | Email |
|---|---|
| Ritu | ritu.dogra.snaatak@mygurukulam.co |

---

# 8. References

| Source | Reference Link |
|---|---|
| **Git Documentation** | [Git Official Documentation](https://git-scm.com/doc) |
| **GitHub Documentation** | [GitHub Documentation](https://docs.github.com/) |
| **Microrepo Strategy Documentation** | [Microrepo Documentation](https://github.com/SnaatakAllStars/Sprint-1/blob/SCRUM-99-RITU/Documentation/VCS_Design/Identify_Devops_Repositories/DOC/README.md)
