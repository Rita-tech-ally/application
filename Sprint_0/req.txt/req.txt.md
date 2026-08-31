<img width="738" height="216" alt="image" src="https://github.com/user-attachments/assets/335edf5b-2f08-4bee-b895-4aaef22c52d6" />

# Documentation for requirements.txt

***Documentation— Python Dependency Management**

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 27/08/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |
---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Prerequisites](#2-prerequisites)
3. [Why requirements.txt is Used](#3-why-requirementstxt-is-used)
4. [File Structure and Syntax](#4-file-structure-and-syntax)
5. [Dependency Versioning Best Practices](#5-dependency-versioning-best-practices)
6. [Conclusion](#6-conclusion)
7. [FAQs](#7-faqs)
8. [Contact Information](#8-contact-information)
9. [References](#9-references)

---

## 1. Introduction

requirements.txt is a standard text file used in Python projects to specify and list all the external dependencies, libraries, and packages required to run a particular application or script. It acts as a blueprint for setting up a consistent development, testing, or production environment.

---

## 2. Prerequisites

Python and pip are required before working with `requirements.txt`.

Verify that Python is installed:

```bash
python3 --version
```

Verify that pip is available:

```bash
pip3 --version
```


It is strongly recommended to use a virtual environment so that dependencies are isolated per project:

```bash
python3 -m venv venv
source venv/bin/activate      # Linux / macOS
venv\Scripts\activate         # Windows
```

----

> **Note:** Always work inside an activated virtual environment before installing or freezing packages, otherwise system-wide packages may get captured or affected.

---

## 3. Why requirements.txt is Used


### 3.1 Why It Is Needed

Without a declared dependency list, every developer or server may end up with a different combination of package versions, which leads to the classic "it works on my machine" problem. `requirements.txt` solves this by acting as a single source of truth for what the project needs.

| Benefit          | Description                                                                |
| ----------------- | --------------------------------------------------------------------------- |
| Reproducibility    | Anyone can recreate the exact same environment using one command.           |
| Portability        | The project can be moved between machines, containers, or cloud environments easily. |
| Collaboration      | Team members and CI/CD systems install identical dependencies.              |
| Version Control    | Dependency changes are tracked in Git alongside code changes.               |
| Automation         | Deployment scripts, Docker images, and pipelines can install dependencies automatically. |

----

### 3.2 Common Use Cases

* Setting up a new developer's local environment quickly.
* Building Docker images with a consistent dependency layer.
* Installing dependencies in CI/CD pipelines (GitHub Actions, Jenkins, GitLab CI).
* Deploying applications to staging or production servers.
* Maintaining separate dependency sets for development, testing, and production.
* Auditing and upgrading project dependencies over time.

---

## 4. File Structure and Syntax

`requirements.txt` is a plain text file, typically placed in the project's root directory, with one requirement specifier per line.

```text
package_name==1.2.3
another_package>=2.0,<3.0
# this is a comment
-r base-requirements.txt
```
----
### 4.1 Version Specifiers

| Operator | Meaning                                    | Example                    |
| -------- | ------------------------------------------- | --------------------------- |
| `==`     | Exact version match                         | `requests==2.31.0`          |
| `>=`     | Minimum version (any newer allowed)         | `numpy>=1.24`                |
| `<=`     | Maximum version allowed                     | `flask<=3.0.0`               |
| `!=`     | Excludes a specific version                 | `django!=4.1.0`              |
| `~=`     | Compatible release (allows patch/minor updates) | `requests~=2.31.0`       |
| `>`, `<` | Strict greater than / less than             | `pandas>1.0,<2.0`             |
| `*`      | Wildcard within a version segment           | `pillow==10.*`                |


---

## 5. Dependency Versioning Best Practices

### 5.1 Pin Exact Versions for Production

For production and deployment files, pin exact versions using `==` so that builds are fully reproducible and unaffected by upstream releases.

```text
gunicorn==22.0.0
```
----

### 5.2 Allow Controlled Flexibility for Libraries

When building a reusable library (not a deployed application), overly strict pins can cause conflicts for consumers. Prefer compatible ranges using `~=` or explicit lower/upper bounds.

```text
requests~=2.31
```
----

### 5.3 Follow Semantic Versioning Awareness

Most Python packages follow `MAJOR.MINOR.PATCH` semantic versioning. Understanding this helps choose safe ranges:

| Segment | Meaning                          | Upgrade Risk                            |
| ------- | ---------------------------------- | ------------------------------------------ |
| MAJOR   | Breaking changes                   | High — review changelog before upgrading   |
| MINOR   | New backward-compatible features   | Low to Medium                              |
| PATCH   | Backward-compatible bug fixes      | Low                                        |

----

### 5.4 Use a Lock File for Full Reproducibility

`pip freeze` captures a flat snapshot but does not record the dependency graph. Tools such as `pip-tools` generate a fully resolved, hash-locked file from a lightweight source file.

```bash
pip install pip-tools
pip-compile requirements.in -o requirements.txt
```

----

### 5.5 Keep Dependencies Minimal and Reviewed

* Only include packages that are actually imported by the project.
* Remove unused packages regularly to reduce the attack surface.
* Avoid pinning to pre-release or nightly builds in production.
* Document why an unusual version constraint exists, using a comment.

----

### 5.6 Keep Dependencies Updated Safely

List outdated packages before upgrading, and upgrade one package at a time where possible:

```bash
pip list --outdated
```
----

### 5.7 Never Commit Secrets or Local Paths

Avoid referencing local file paths or private credentials directly inside `requirements.txt`. Use private package indexes or environment variables instead.

> **Note:** Treat `requirements.txt` as part of the codebase: review changes to it in pull requests just like application code.

---

## 6. Conclusion

A requirements.txt file ensures consistent and reproducible Python dependencies across environments when paired with proper versioning. Before production deployment, always test the installation in a clean virtual environment, review constraints, check for vulnerabilities, and track the file in version control. Regularly updating dependencies is also essential to maintain long-term security.

---

## 7. FAQs

**What is requirements.txt?**
It is a plain text file that lists the Python packages, and optionally their versions, that a project depends on, used by pip to install those dependencies.

**How do I generate requirements.txt?**

```bash
pip freeze > requirements.txt
```

**How do I install packages from requirements.txt?**

```bash
pip install -r requirements.txt
```

**Should I pin exact versions?**
For applications and production deployments, yes — exact pins keep builds reproducible. For shared libraries, looser, well-considered ranges are usually preferred.

**How do I remove a package from requirements.txt?**
Uninstall it with `pip uninstall <package>`, then remove or regenerate the corresponding line in `requirements.txt`.

**How can I keep dependencies secure?**
Run `pip-audit` periodically, subscribe to Dependabot or Renovate alerts, and review changelogs before upgrading major versions.

---

## 8. Contact Information

| Name | Email Address |
| ---- | -------------- |
| Ritu |                |

---

## 9. References

| Links                                                                                   | Description                                             |
| ------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| https://pip.pypa.io/en/stable/reference/requirements-file-format/                         | Official pip requirements file format reference.           |
| https://packaging.python.org/en/latest/discussions/install-requires-vs-requirements/       | Python Packaging Authority guide on requirements files.    |
| https://pip-tools.readthedocs.io/en/latest/                                                | pip-tools documentation for compiling locked requirements. |
| https://semver.org/                                                                       | Semantic Versioning specification.                          |
| https://pypi.org/project/pip-audit/                                                       | pip-audit documentation for vulnerability scanning.         |
