# SOP: Common Stack | Applications | JQ | Installation

---

# Author Table

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 29/08/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |

---

# Table of Contents

| **S. No.** | **Section**            | **Description**                            |
| ---------: | ---------------------- | ------------------------------------------ |
|          1 | Introduction           | Overview of JQ and its purpose             |
|          2 | Purpose                | Objective and scope of the SOP             |
|          3 | Prerequisites          | Required OS, access, packages, and network |
|          4 | Update System Packages | Update the Ubuntu package repository       |
|          5 | Install JQ             | Install JQ using APT                       |
|          6 | Verify JQ Installation | Check installed JQ version                 |
|          7 | Validation             | Test JQ with sample JSON data              |
|          8 | Use Cases              | Common JQ commands and DevOps use cases    |
|          9 | Troubleshooting        | Common installation and execution issues   |
|         10 | Best Practices         | Recommended operational practices          |
|         11 | Conclusion             | Summary of the installation procedure      |
|         12 | Contact Information    | SOP owner and contact details              |
|         13 | References             | Official and supporting documentation      |

---

# 1. Introduction

This SOP provides a structured guide to installing **JQ**, a lightweight command-line JSON processor, on an Ubuntu/Linux environment.

JQ is commonly used in DevOps environments to **parse, filter, transform, and extract data from JSON files and API responses**.

---

# 2. Purpose

The purpose of this SOP is to provide a standardized procedure for:

* Installing JQ on Ubuntu/Linux.
* Verifying the JQ installation.
* Processing and querying JSON data.
* Preparing JQ for DevOps automation and scripting.

These procedures help maintain **system stability, reliability, performance, and operational consistency**.

---

# 3. Prerequisites

| **Prerequisite**  | **Details**                                    |
| ----------------- | ---------------------------------------------- |
| OS & Access       | Ubuntu/Linux with terminal access              |
| Required Package  | `jq`                                           |
| Required Commands | `apt`, `jq`                                    |
| Permissions       | `sudo/root access`                             |
| Network           | Internet connectivity for package installation |
| Repository        | Ubuntu APT repository                          |

---

# 4. Update System Packages

## Step 1: Update Package Repository

Update the local package index before installing JQ.

```bash
sudo apt update
```

## Step 2: Upgrade Packages

Optionally upgrade installed packages.

```bash
sudo apt upgrade -y
```

<details>
<summary>:camera_with_flash: <strong>Screenshot - System Package Update</strong></summary>

<img width="800" height="450" alt="System Package Update" src="<SCREENSHOT_01_URL>" />

</details>

---

# 5. Install JQ

## Step 1: Install JQ

Install JQ using the Ubuntu APT package manager.

```bash
sudo apt install jq -y
```

The package manager downloads and installs JQ along with any required dependencies.

<details>
<summary>:camera_with_flash: <strong>Screenshot - JQ Installation</strong></summary>

<img width="800" height="450" alt="JQ Installation" src="<SCREENSHOT_02_URL>" />

</details>

## Step 2: Locate JQ

Verify the installed executable path.

```bash
which jq
```

Expected output:

```text
/usr/bin/jq
```

---

# 6. Verify JQ Installation

## Step 1: Check JQ Version

Run:

```bash
jq --version
```

Expected output will be similar to:

```text
jq-1.6
```

The exact version may vary depending on the Ubuntu release and repository package version.

<details>
<summary>:camera_with_flash: <strong>Screenshot - JQ Version Verification</strong></summary>

<img width="800" height="450" alt="JQ Version Verification" src="<SCREENSHOT_03_URL>" />

</details>

## Step 2: Check JQ Help

```bash
jq --help
```

This confirms that the JQ command is available and executable.

---

# 7. Validation

## Step 1: Create Sample JSON

Run:

```bash
echo '{"name":"Yogesh","role":"DevOps","tool":"JQ"}' > sample.json
```

## Step 2: Display JSON

```bash
cat sample.json
```

Expected:

```json
{"name":"Yogesh","role":"DevOps","tool":"JQ"}
```

## Step 3: Parse JSON Using JQ

Run:

```bash
jq '.' sample.json
```

Expected:

```json
{
  "name": "Yogesh",
  "role": "DevOps",
  "tool": "JQ"
}
```

## Step 4: Extract a Specific Value

Run:

```bash
jq '.name' sample.json
```

Expected:

```text
"Yogesh"
```

<details>
<summary>:camera_with_flash: <strong>Screenshot - JQ JSON Validation</strong></summary>

<img width="800" height="450" alt="JQ JSON Validation" src="<SCREENSHOT_04_URL>" />

</details>

### Final Validation Checklist

| **Validation**  | **Expected Result**            |
| --------------- | ------------------------------ |
| JQ installation | JQ installed successfully      |
| JQ executable   | `/usr/bin/jq` available        |
| JQ version      | Version displayed successfully |
| JSON parsing    | JSON formatted successfully    |
| JSON extraction | Required JSON value displayed  |

---

# 8. Use Cases

| **Scenario**         | **Commands / Actions**      |                                       |
| -------------------- | --------------------------- | ------------------------------------- |
| Check JQ version     | `jq --version`              |                                       |
| Format JSON          | `jq '.' file.json`          |                                       |
| Extract a value      | `jq '.name' file.json`      |                                       |
| Extract nested value | `jq '.user.name' file.json` |                                       |
| Filter JSON data     | `jq '.[]                    | select(.status=="active")' file.json` |
| Process API response | `curl <API_URL> \| jq '.'`  |                                       |

---

# 9. Troubleshooting

| **Issue**               | **Cause**                         | **Solution**                            |
| ----------------------- | --------------------------------- | --------------------------------------- |
| `jq: command not found` | JQ is not installed               | Run `sudo apt install jq -y`            |
| APT cannot find JQ      | Package repository is not updated | Run `sudo apt update`                   |
| Permission denied       | Insufficient permissions          | Use `sudo` where required               |
| Invalid JSON            | Incorrect JSON syntax             | Validate and correct the JSON structure |
| Empty output            | Incorrect JQ filter               | Verify the JSON structure and JQ query  |

---

# 10. Best Practices

| **Best Practice**    | **Description**                                                     |
| -------------------- | ------------------------------------------------------------------- |
| Use APT for Ubuntu   | Install JQ through the official Ubuntu package repository.          |
| Verify installation  | Always run `jq --version` after installation.                       |
| Validate JSON        | Check JSON syntax before applying complex filters.                  |
| Use specific filters | Extract only the required fields from large JSON responses.         |
| Use in automation    | Integrate JQ into Bash scripts, CI/CD pipelines, and API workflows. |

---

# 11. Conclusion

This SOP provides a standardized approach to installing and validating **JQ on Ubuntu/Linux**.

Following these procedures helps DevOps engineers reliably use JQ for **JSON parsing, data extraction, API response processing, automation, and CI/CD workflows**.

---

# 12. Contact Information

| **Name** | **Email** |
| -------- | --------- |
| Yogesh   | <email>   |

---

# 13. References

| **Topic**                                                 | **Description**                                         |
| --------------------------------------------------------- | ------------------------------------------------------- |
| [JQ Documentation](https://jqlang.org/manual/)            | Official JQ manual and usage reference.                 |
| [JQ GitHub](https://github.com/jqlang/jq)                 | JQ source code and project information.                 |
| [Ubuntu Documentation](https://documentation.ubuntu.com/) | Ubuntu administration and package management reference. |
