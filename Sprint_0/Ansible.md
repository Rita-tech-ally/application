<img width="554" height="361" alt="image" src="https://github.com/user-attachments/assets/0b0374f5-9eb8-4881-bfee-c381e13fd504" />

# Documentation for Ansible Role

## Document Information



| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 28/08/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |
---

# Table of Contents

1. [Introduction](#1-introduction)
2. [What is an Ansible Role](#2-what-is-an-ansible-role)
3. [Why Ansible Roles are Used](#3-why-ansible-roles-are-used)
4. [Features of Ansible Roles](#4-features-of-ansible-roles)
5. [Benefits of Ansible Roles](#5-benefits-of-ansible-roles)
6. [Conclusion](#6-conclusion)
7. [FAQs](#7-faqs)
8. [Contact Information](#8-contact-information)
9. [References](#9-references)

---

## 1. Introduction

An Ansible Role organizes complex automation code by splitting single, large playbooks into structured, reusable directories. This makes your code easier to read, maintain, and share.

Primary Uses:

Setup: Installing software, creating users, and configuring infrastructure.

Management: Controlling services, configuration files, and monitoring tools.

Deployment: Rolling out applications.

## 2. What is an Ansible Role

An **Ansible Role** is a standardized directory structure used to organize related automation tasks and supporting files.

A role can contain:

* **Tasks** – Define the actions Ansible should perform.
* **Handlers** – Perform actions when notified by tasks, such as restarting a service.
* **Defaults** – Store default variable values.
* **Vars** – Store role-specific variables.
* **Templates** – Store Jinja2 templates for dynamic configuration files.
* **Files** – Store static files that need to be copied to managed servers.
* **Meta** – Define role dependencies and metadata.

---
## Why Use Roles?
* **Modularity**: Break large, complex playbooks into small, manageable chunks.
* **Reusability**: Write a "MySQL" role once, and use it across 10 different projects.
* **Structure**: Enforces a standard layout that any Ansible developer can instantly understand.
* **Sharing** : Roles can be shared via Ansible Galaxy (the public repository for Ansible roles).

<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/e44e477f-c768-4c6b-bb9f-60b4e3375c8a" />

---

## Ansible Role
The magic of Roles lies in their specific directory structure. Ansible looks for a main.yml file in each of these standard directories to know what to do.

To create the skeleton of a role automatically, use the ansible-galaxy command:

ansible-galaxy init role


<img width="214" height="418" alt="image" src="https://github.com/user-attachments/assets/69d3f3ce-900f-4fdc-8b23-35f99887927b" />

## 4. Features of Ansible Roles

| Feature                      | Description                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------- |
| **Reusable**                 | Roles can be reused across multiple playbooks and servers.                      |
| **Standard Structure**       | Roles follow a predefined directory structure.                                  |
| **Variable Support**         | Variables can be managed using `defaults` and `vars`.                           |
| **Template Support**         | Jinja2 templates can dynamically generate configuration files.                  |
| **Handler Support**          | Services can be restarted or reloaded when configuration changes.               |
| **Dependency Management**    | Roles can define dependencies using `meta/main.yml`.                            |
| **Easy Maintenance**         | Individual components can be updated without changing the complete automation.  |
| **Scalable**                 | Roles can be used as projects grow from a few servers to large infrastructures. |
| **Version Control Friendly** | Roles can be maintained and reviewed using Git.                                 |

---

## 5. Benefits of Ansible Roles

Ansible Roles provide several benefits for infrastructure and application automation:

* Reduce duplicate automation code.
* Improve code organization.
* Make automation easier to understand.
* Support reuse across different environments.
* Simplify troubleshooting and maintenance.
* Improve team collaboration.
* Make large Ansible projects easier to scale.
* Provide a standard and predictable project structure.

### Example

Without a role, a playbook may contain all installation, configuration, template, service, and variable logic in one place.

With a role, the same automation can be organized as:

```text
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    ├── defaults/
    │   └── main.yml
    └── files/
```

This separation makes the automation **cleaner, reusable, maintainable, and easier to scale**.

---

## 6. Conclusion

Ansible roles organize automation into a structured directory of tasks, variables, and handlers, making it reusable, modular, maintainable, and scalable across environments. Following this standard structure simplifies development, testing, troubleshooting, and team collaboration.

---

## 7. FAQs

###  What is an Ansible Role?

An Ansible Role is a predefined directory structure used to organize and reuse Ansible automation code, including tasks, variables, handlers, templates, files, and dependencies.

### Why are Ansible Roles used?

Ansible Roles are used to make automation code **reusable, modular, organized, maintainable, and scalable**.

### What are the main directories of an Ansible Role?

The commonly used directories are:

```text
tasks/
handlers/
defaults/
vars/
templates/
files/
meta/
```

### What is `tasks/main.yml` used for?

`tasks/main.yml` contains the main tasks that Ansible executes when the role is applied.

###  What is the purpose of `handlers/main.yml`?

Handlers are used for actions that should run when they are notified by a task, such as restarting or reloading a service after a configuration change.

###  What is the difference between `defaults` and `vars`?

`defaults/main.yml` is generally used for default variable values that can be easily overridden.

`vars/main.yml` is used for role-specific variables that normally have higher precedence and are intended to be less frequently overridden.

### What is the purpose of the `templates` directory?

The `templates` directory stores Jinja2 template files used to generate dynamic configuration files based on variables.

---

## 8. Contact Information

| Name |            Email Address         |
| ---- | ---------------------------------|
| Ritu | ritu.dogra.snaatak@mygurukulam.co|

---

## 9. References

| Reference                                                                                                        | Description                                                           |
| ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| [Ansible Documentation](https://docs.ansible.com/)                                                               | Official Ansible documentation and user guide.                        |
| [Ansible Roles Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html) | Official documentation for creating and using Ansible Roles.          |
| [Ansible Galaxy](https://galaxy.ansible.com/)                                                                    | Repository for discovering and sharing Ansible Roles and Collections. |
| [Ansible Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)           | Official Ansible tips and best practices.                             |

```
```
