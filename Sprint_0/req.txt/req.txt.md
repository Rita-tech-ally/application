# Documentation for requirements.txt

<img width="738" height="216" alt="image" src="https://github.com/user-attachments/assets/335edf5b-2f08-4bee-b895-4aaef22c52d6" />

***Documentation— Python Dependency Management**

## Document Information


| Author       | Created On | Version | Last Updated By | Last Edited On | | L0 Reviewer | L1 Reviewer | L2 Reviewer  |
| ------------ | ---------- | ------- | --------------- | -------------- | ------------ | ----------- | ----------- | ------------ |
| Ritu | 27/08/2026 | 1.0     | Ritu  | 31/08/2026     |      |Liyakhat |Aman Raj |Sandeep Rawat/Ravindra |
|
---

## Table of Contents

1. [Purpose](#1-purpose)
2. [Prerequisites](#2-prerequisites)
3. [Why requirements.txt is Used](#3-why-requirementstxt-is-used)
4. [File Structure and Syntax](#4-file-structure-and-syntax)
5. [Configuration Steps](#5-configuration-steps)
6. [Dependency Versioning Best Practices](#7-dependency-versioning-best-practices)
7. [Monitoring and Troubleshooting](#9-monitoring-and-troubleshooting)
8. [Conclusion](#10-conclusion)
9. [FAQs](#11-faqs)
10. [Contact Information](#12-contact-information)
11. [References](#13-references)

---

## 1. Purpose

Python projects rely on external libraries, and different environments (development, testing, staging, production) must install the exact same set of dependencies to behave consistently. The `requirements.txt` file is the de-facto standard mechanism for declaring, sharing, and reproducing the Python packages a project needs.

`requirements.txt` can be used to:

* Declare the exact packages a project depends on.
* Reproduce an identical environment across machines and environments.
* Share dependency information with other developers and CI/CD pipelines.
* Pin package versions to avoid unexpected breaking changes.
* Separate dependencies for development, testing, and production.
* Automate environment setup during deployment and containerization.

This SOP explains why `requirements.txt` is used, common use cases, file structure and syntax, dependency versioning best practices, configuration steps, testing, automation, and troubleshooting.

---

## 2. Prerequisites

Python and pip are required before working with `requirements.txt`.

Verify that Python is installed:

```bash
python3 --version
```
<img width="794" height="47" alt="Screenshot from 2026-08-27 18-32-54" src="https://github.com/user-attachments/assets/d175e07b-4287-4881-8993-c3cfed6eafdb" />

Verify that pip is available:

```bash
pip3 --version
```
<img width="894" height="58" alt="Screenshot from 2026-08-27 18-34-31" src="https://github.com/user-attachments/assets/40cc98f2-1bed-4531-b4fe-23690fd6c9f9" />

It is strongly recommended to use a virtual environment so that dependencies are isolated per project:

```bash
python3 -m venv venv
source venv/bin/activate      # Linux / macOS
venv\Scripts\activate         # Windows
```
<img width="891" height="42" alt="image" src="https://github.com/user-attachments/assets/6a7f6821-bf6e-47ef-b8b7-88424ef924df" />

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

### 4.2 Other Common Syntax Elements

| Syntax        | Purpose                                | Example                                             |
| -------------- | ---------------------------------------- | ---------------------------------------------------- |
| `#`            | Comment line, ignored by pip             | `# core dependencies`                                |
| `-r`           | Include another requirements file        | `-r base.txt`                                        |
| `-e`           | Editable / local install                 | `-e .`                                                |
| `[extra]`      | Install optional extras of a package     | `uvicorn[standard]==0.30.0`                           |
| `package @ url`| Install from a direct URL                | `mypkg @ git+https://github.com/org/mypkg.git`        |
| `;`            | Environment marker (conditional install) | `pywin32==306; sys_platform=='win32'`                 |

---

## 5. Configuration Steps

| Step | Action                              | Command                                             | Description                                       |
| ---- | ------------------------------------ | ----------------------------------------------------- | ---------------------------------------------------- |
| 1    | Create/activate a virtual environment | `python3 -m venv venv && source venv/bin/activate`   | Isolates project dependencies.                       |
| 2    | Install required packages            | `pip install flask requests`                          | Installs the packages needed for the project.        |
| 3    | Generate requirements.txt            | `pip freeze > requirements.txt`                       | Captures installed packages and exact versions.       |
| 4    | Review and edit the file             | `nano requirements.txt`                                | Remove unnecessary or environment-specific packages.  |
| 5    | Install from requirements.txt        | `pip install -r requirements.txt`                      | Recreates the environment on another machine.         |

### 5.1 Generating requirements.txt

After installing the packages a project needs, capture the exact installed versions:

```bash
pip freeze > requirements.txt
```
<img width="897" height="257" alt="Screenshot from 2026-08-27 18-37-25" src="https://github.com/user-attachments/assets/2215a3b7-774e-4b2a-9980-108fcb5a3868" />

### 5.2 Installing From requirements.txt

On a new machine or environment, install all listed dependencies in one step:

```bash
pip install -r requirements.txt
```
<img width="1231" height="232" alt="Screenshot from 2026-08-27 18-38-18" src="https://github.com/user-attachments/assets/25ea94f6-785c-4e9e-b29b-4a732e5283e6" />

### 5.3 Separating Dependencies by Environment

Larger projects commonly split dependencies into multiple files:

```text
requirements.txt          # production dependencies
requirements-dev.txt      # development/test dependencies (includes -r requirements.txt)
```

`requirements-dev.txt` example:

```text
-r requirements.txt
pytest==8.2.0
black==24.4.2
flake8==7.0.0
```

---

## 6. Dependency Versioning Best Practices

### 6.1 Pin Exact Versions for Production

For production and deployment files, pin exact versions using `==` so that builds are fully reproducible and unaffected by upstream releases.

```text
gunicorn==22.0.0
```

### 6.2 Allow Controlled Flexibility for Libraries

When building a reusable library (not a deployed application), overly strict pins can cause conflicts for consumers. Prefer compatible ranges using `~=` or explicit lower/upper bounds.

```text
requests~=2.31
```

### 6.3 Follow Semantic Versioning Awareness

Most Python packages follow `MAJOR.MINOR.PATCH` semantic versioning. Understanding this helps choose safe ranges:

| Segment | Meaning                          | Upgrade Risk                            |
| ------- | ---------------------------------- | ------------------------------------------ |
| MAJOR   | Breaking changes                   | High — review changelog before upgrading   |
| MINOR   | New backward-compatible features   | Low to Medium                              |
| PATCH   | Backward-compatible bug fixes      | Low                                        |

### 6.4 Use a Lock File for Full Reproducibility

`pip freeze` captures a flat snapshot but does not record the dependency graph. Tools such as `pip-tools` generate a fully resolved, hash-locked file from a lightweight source file.

```bash
pip install pip-tools
pip-compile requirements.in -o requirements.txt
```

### 6.5 Keep Dependencies Minimal and Reviewed

* Only include packages that are actually imported by the project.
* Remove unused packages regularly to reduce the attack surface.
* Avoid pinning to pre-release or nightly builds in production.
* Document why an unusual version constraint exists, using a comment.

### 6.6 Keep Dependencies Updated Safely

List outdated packages before upgrading, and upgrade one package at a time where possible:

```bash
pip list --outdated
```

### 6.7 Never Commit Secrets or Local Paths

Avoid referencing local file paths or private credentials directly inside `requirements.txt`. Use private package indexes or environment variables instead.

> **Note:** Treat `requirements.txt` as part of the codebase: review changes to it in pull requests just like application code.

---



## 7. Monitoring and Troubleshooting

### 7.1 Check Installed vs Declared Packages

```bash
pip freeze
```

### 7.2 Audit for Known Vulnerabilities

Use `pip-audit` to scan installed packages against known vulnerability databases:

```bash
pip install pip-audit
pip-audit -r requirements.txt
```

### 7.3 Common Issues

| Issue                                   | Cause                                                     | Solution                                                        |
| ------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------------- |
| Dependency conflict during install         | Two packages require incompatible versions of the same dependency | Run `pip check`; loosen or align version constraints.               |
| "Works on my machine"                      | Packages installed manually without updating requirements.txt | Regenerate with `pip freeze` after every change; commit the update. |
| Package not found                          | Typo in package name or private index not configured         | Verify the package name on PyPI; add `--index-url` if using a private repository. |
| Wrong Python version behaviour              | Environment markers not used for platform-specific packages  | Add markers such as `; sys_platform=='win32'`.                       |
| Slow or failing CI installs                | No caching of pip downloads                                   | Cache the pip directory between CI runs.                             |
| Silent version drift over time             | Ranges too loose (e.g. only `>=`)                              | Pin exact versions for production; use a lock file.                  |

---

## 8. Conclusion

A requirements.txt file ensures consistent and reproducible Python dependencies across environments when paired with proper versioning. Before production deployment, always test the installation in a clean virtual environment, review constraints, check for vulnerabilities, and track the file in version control. Regularly updating dependencies is also essential to maintain long-term security.

---

## 9. FAQs

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

## 10. Contact Information

| Name | Email Address |
| ---- | -------------- |
| Ritu |                |

---

## 11. References

| Links                                                                                   | Description                                             |
| ------------------------------------------------------------------------------------------ | ---------------------------------------------------------- |
| https://pip.pypa.io/en/stable/reference/requirements-file-format/                         | Official pip requirements file format reference.           |
| https://packaging.python.org/en/latest/discussions/install-requires-vs-requirements/       | Python Packaging Authority guide on requirements files.    |
| https://pip-tools.readthedocs.io/en/latest/                                                | pip-tools documentation for compiling locked requirements. |
| https://semver.org/                                                                       | Semantic Versioning specification.                          |
| https://pypi.org/project/pip-audit/                                                       | pip-audit documentation for vulnerability scanning.         |
