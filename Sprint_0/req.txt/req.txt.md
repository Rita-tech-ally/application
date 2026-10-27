# Jenkins Shared Library for Golang Dependency Scanning (SCA)

<p align="center">
  <img width="333" height="155" alt="Golang SCA Pipeline" src="https://github.com/user-attachments/assets/be9907e0-f42a-4213-af2f-15eb5e0aacd7" />
</p>

---

## Document Information

| **Property** | **Details** |
|---|---|
| **Author** | Ritu |
| **Created On** | 24/10/2026 |
| **Version** | 1.0 |
| **L0 Reviewer** | Liyakhat / Anirudh |
| **L1 Reviewer** | Aman Raj |
| **L2 Reviewer** | Sandeep Rawat / Ravindra |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Library Type](#2-library-type)
3. [Library Workflow](#3-library-workflow)
4. [Prerequisites and Dependencies](#4-prerequisites-and-dependencies)
5. [Implementation Details](#5-implementation-details)
6. [Notifications](#6-notifications)
7. [Error Handling and Cleanup](#7-error-handling-and-cleanup)
8. [Conclusion](#8-conclusion)
9. [Contact Information](#9-contact-information)
10. [References](#10-references)

---

## 1. Introduction

This Jenkins Shared Library automates Software Composition Analysis (SCA) for Go applications using `govulncheck`. It prepares dependencies, scans source code for known vulnerabilities, generates and archives reports, and sends Slack and email notifications.

## 2. Library Type

| **Property** | **Details** |
|---|---|
| **Library Type** | Jenkins Shared Library |
| **Language** | Groovy |
| **Application** | Go |
| **Analysis Type** | Software Composition Analysis (SCA) |
| **Scanning Tool** | `govulncheck` |
| **Dependency Management** | Go Modules |
| **Source Control** | GitHub |
| **Notifications** | Slack and Email |
| **Report Storage** | Jenkins Artifacts |

## 3. Library Workflow

```text
Start Pipeline
      |
      v
Checkout Employee API
      |
      v
Call goDependencyScan()
      |
      v
Install govulncheck
      |
      v
Prepare Go Dependencies
      |
      v
Scan Source Code
      |
      v
Generate and Archive Report
      |
      v
Send Notifications
      |
      v
Clean Workspace
      |
      v
End Pipeline
```

## 4. Prerequisites and Dependencies

### 4.1 Jenkins Requirements

| **Requirement** | **Purpose** |
|---|---|
| **Jenkins Pipeline Plugin** | Pipeline execution |
| **Git Plugin** | Repository checkout |
| **Configured Shared Library** | Reusable scanning functions |
| **Slack Plugin** | `slackSend()` notifications |
| **Email Extension Plugin** | `emailext()` notifications |
| **Workspace Cleanup Plugin** | `cleanWs()` cleanup |

### 4.2 Build Agent Requirements

- Go and Git installed.
- Network access to GitHub and Go module repositories.
- GitHub credentials configured in Jenkins.
- Permission to create reports and archive build artifacts.

## 5. Implementation Details

### 5.1 Configure Shared Library

```groovy
library identifier: 'Shared_Library@SCRUM-296-RITU',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        remote: 'https://github.com/SnaatakAllStars/Shared_Library.git',
        credentialsId: 'Github-cred'
    ])
```

Loads reusable functions from the configured shared library branch.

### 5.2 Checkout Application Repository

```groovy
stage('Checkout Code') {
    steps {
        dir("${env.APP_REPO_NAME}") {
            git branch: env.APP_BRANCH,
                credentialsId: env.GIT_CREDS_ID,
                url: env.APP_REPO_URL
        }
    }
}
```

| **Variable** | **Value** |
|---|---|
| **`APP_REPO_NAME`** | `Employee-api` |
| **`APP_REPO_URL`** | `https://github.com/SnaatakAllStars/employee-api.git` |
| **`APP_BRANCH`** | `main` |
| **`GIT_CREDS_ID`** | `Github-cred` |

### 5.3 Execute Dependency Scan

```groovy
stage('Dependency Scanning (SCA)') {
    steps {
        goDependencyScan(
            appDir: env.APP_REPO_NAME
        )
    }
}
```

The `goDependencyScan()` function runs the shared scanning workflow for the application directory.

### 5.4 Install govulncheck

```bash
set -eu
mkdir -p "$GO_BIN"
export GOBIN="$GO_BIN"
go install golang.org/x/vuln/cmd/govulncheck@latest
"$GO_BIN/govulncheck" -version
```

Installs `govulncheck` in the configured binary directory and verifies the installation. Using `@latest` may install a different version over time.

### 5.5 Prepare Dependencies

```bash
set -eu
go mod tidy
go mod download
```

- `go mod tidy` updates module definitions and checksums as needed.
- `go mod download` downloads project dependencies.

**Note:** `go mod tidy` may modify `go.mod` and `go.sum`.

### 5.6 Run Vulnerability Scan

```bash
"$GO_BIN/govulncheck" \
    -scan package \
    -show verbose \
    ./... > reports/govulncheck-report.txt 2>&1
```

Scans Go packages and saves the output and errors to the report file.

| **Option** | **Purpose** |
|---|---|
| **`-scan package`** | Scans packages for relevant vulnerabilities |
| **`-show verbose`** | Includes additional scan details |
| **`./...`** | Targets packages in the current module |
| **`> report 2>&1`** | Redirects output and errors to a file |

The shared library handles the scanner exit status:

- **Exit code `0`:** Scan completed without a scanner-reported vulnerability finding.
- **Exit code `3`:** Treated by the script as a completed scan; review the report for findings.
- **Other exit codes:** Treated as scan failures.

A completed scan does not guarantee that the application is vulnerability-free.

### 5.7 Archive Report

```groovy
archiveArtifacts(
    artifacts: 'reports/govulncheck-report.txt',
    allowEmptyArchive: true
)
```

Archives the vulnerability report on the Jenkins build page for later review.

## 6. Notifications

The `notifyStatus.groovy` function handles Slack and email notifications.

| **Notification** | **Details** |
|---|---|
| **Slack** | Build status, job name, build number, and build URL |
| **Email** | Build status, job details, console URL, and report link |

- `slackSend()` sends Slack notifications.
- `emailext()` sends email notifications.
- Separate error handling allows each notification attempt to be handled independently.

## 7. Error Handling and Cleanup

| **Mechanism** | **Purpose** |
|---|---|
| **`set -eu`** | Stops shell execution on errors or unset variables |
| **Exit-code handling** | Identifies accepted scan completion codes and failures |
| **`try-catch`** | Handles scanning exceptions |
| **`archiveArtifacts()`** | Archives the report when available |
| **Notification error handling** | Handles Slack and email errors separately |
| **`post { always }`** | Runs final pipeline actions |
| **`cleanWs()`** | Cleans workspace files |

## 8. Conclusion

This shared library automates Go dependency scanning with `govulncheck`, report generation, artifact archiving, notifications, error handling, and workspace cleanup.

## 9. Contact Information

| **Property** | **Details** |
|---|---|
| **Name** | Ritu |
| **Email** | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

## 10. References

- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [Go Documentation](https://go.dev/doc/)
- [Go Modules Reference](https://go.dev/ref/mod)
- [govulncheck Documentation](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)
- [Go Vulnerability Database](https://vuln.go.dev/)
