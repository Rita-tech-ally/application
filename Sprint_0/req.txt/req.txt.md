# Jenkins Shared Library for Golang Dependency Scanning (SCA)

<p align="center">
  <img width="333" height="155" alt="Golang SCA Pipeline" src="https://github.com/user-attachments/assets/be9907e0-f42a-4213-af2f-15eb5e0aacd7" />
</p>

---

## Document Information

| Property | Details |
|---|---|
| **Author** | Ritu |
| **Created On** | 24/10/2026 |
| **Version** | 1.0 |
| **L0 Reviewer** | Liyakhat / Anirudh |
| **L1 Reviewer** | Aman Raj |
| **L2 Reviewer** | Sandeep Rawat / Ravindra |

---

## Table of Contents

1. Introduction
2. Library Type
3. Library Workflow
4. Prerequisites and Dependencies
5. Implementation Details
6. Notifications
7. Error Handling and Cleanup
8. Conclusion
9. Contact Information
10. References

---

## 1. Introduction

This Jenkins Shared Library automates Software Composition Analysis (SCA) for Go-based applications. It uses `govulncheck` to scan Go source code for known vulnerabilities, prepares dependencies, generates a vulnerability report, archives the report in Jenkins, and sends Slack and email notifications.

## 2. Library Type

| Property | Details |
|---|---|
| **Library Type** | Jenkins Shared Library |
| **Language** | Groovy |
| **Application Type** | Go-based applications |
| **Security Analysis** | Software Composition Analysis (SCA) |
| **Scanning Tool** | `govulncheck` |
| **Dependency Management** | Go Modules |
| **CI Platform** | Jenkins |
| **Source Control** | GitHub |
| **Notifications** | Slack and Email |
| **Report Management** | Jenkins Archived Artifacts |

## 3. Library Workflow

```text
Start Pipeline
      |
      v
Checkout Employee API Repository
      |
      v
Call goDependencyScan()
      |
      v
Install govulncheck
      |
      v
Download Go Dependencies
      |
      v
Scan Go Source Code
      |
      v
Generate Vulnerability Report
      |
      v
Archive Scan Report
      |
      v
Determine Build Status
      |
      v
Send Slack and Email Notifications
      |
      v
Clean Jenkins Workspace
      |
      v
End Pipeline
```

## 4. Prerequisites and Dependencies

### 4.1 Jenkins Requirements

| Requirement | Purpose |
|---|---|
| **Jenkins** | Executes the pipeline |
| **Pipeline Plugin** | Supports pipeline execution |
| **Git Plugin** | Checks out application code |
| **Configured Shared Library** | Provides reusable scanning functions |
| **Slack Notification Plugin** | Provides the `slackSend()` step |
| **Email Extension Plugin** | Provides the `emailext()` step |
| **Workspace Cleanup Plugin** | Provides the `cleanWs()` step |
| **Artifact Archiving Support** | Stores vulnerability reports |

### 4.2 Build Agent Requirements

| Dependency | Purpose |
|---|---|
| **Go** | Executes Go commands and manages dependencies |
| **Git** | Retrieves application source code |
| **`govulncheck`** | Scans Go code for known vulnerabilities |
| **Network Connectivity** | Accesses GitHub and Go module repositories |

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

This loads reusable library functions from the `SCRUM-296-RITU` branch using the configured GitHub credentials.

### 5.2 Checkout Application Repository

```groovy
stage('Checkout Code') {
    steps {
        echo "[INFO] Cloning ${env.APP_REPO_NAME}..."
        dir("${env.APP_REPO_NAME}") {
            git branch: env.APP_BRANCH,
                credentialsId: env.GIT_CREDS_ID,
                url: env.APP_REPO_URL
        }
    }
}
```

| Variable | Value |
|---|---|
| **`APP_REPO_NAME`** | `Employee-api` |
| **`APP_REPO_URL`** | `https://github.com/SnaatakAllStars/employee-api.git` |
| **`APP_BRANCH`** | `main` |
| **`GIT_CREDS_ID`** | `Github-cred` |

This checks out the Employee API repository into the `Employee-api` workspace directory.

### 5.3 Run Go Dependency Scan

```groovy
stage('Dependency Scanning (SCA)') {
    steps {
        goDependencyScan(
            appDir: env.APP_REPO_NAME
        )
    }
}
```

The `goDependencyScan()` function invokes the shared scanning workflow for the specified application directory.

### 5.4 Install govulncheck

```bash
set -eu
mkdir -p "$GO_BIN"
export GOBIN="$GO_BIN"
go install golang.org/x/vuln/cmd/govulncheck@latest
"$GO_BIN/govulncheck" -version
```

- Creates a local Go binary directory.
- Installs `govulncheck` in that directory.
- Verifies the installed version.

**Note:** `@latest` installs the latest available version, so the installed version may change over time.

### 5.5 Download Project Dependencies

```bash
set -eu
go mod tidy
go mod download
```

- `go mod tidy` updates module definitions and checksums as needed.
- `go mod download` downloads Go modules required by the project.

**Note:** `go mod tidy` can modify `go.mod` and `go.sum` in the workspace.

### 5.6 Run Vulnerability Scan

```bash
"$GO_BIN/govulncheck" \
    -scan package \
    -show verbose \
    ./... > reports/govulncheck-report.txt 2>&1
```

This scans the Go project packages and redirects standard output and error output to `reports/govulncheck-report.txt`.

| Option | Purpose |
|---|---|
| **`-scan package`** | Scans packages for relevant vulnerabilities |
| **`-show verbose`** | Displays additional scan details |
| **`./...`** | Targets packages in the current module |
| **`> report 2>&1`** | Saves output and errors to the report file |

The shared library handles the scanner's exit status:

- Exit code `0`: scan completed without a scanner-reported vulnerability finding.
- Exit code `3`: treated by the script as a completed scan; review the report for findings.
- Other exit codes: treated as scan failures.

A completed scan does not guarantee that the application is vulnerability-free. Review the generated report for findings.

### 5.7 Archive Vulnerability Report

```groovy
archiveArtifacts(
    artifacts: 'reports/govulncheck-report.txt',
    allowEmptyArchive: true
)
```

This archives the vulnerability report in Jenkins so it can be accessed from the build page.

## 6. Notifications

The `notifyStatus.groovy` function sends build notifications through Slack and email.

| Notification | Details |
|---|---|
| **Slack** | Build status, job name, build number, and build URL |
| **Email** | Build status, pipeline stage, job/build number, console URL, and report artifact URL |

- `slackSend()` sends Slack notifications.
- `emailext()` sends email notifications.
- Separate error handling helps prevent notification failures from interfering with other notification attempts.

## 7. Error Handling and Cleanup

| Mechanism | Purpose |
|---|---|
| **`set -eu`** | Stops shell execution when a command fails or an unset variable is used |
| **Scan exit-code handling** | Distinguishes accepted scan completion codes from scanner failures |
| **`try-catch`** | Handles scanning exceptions |
| **`archiveArtifacts()`** | Attempts to preserve the vulnerability report |
| **Slack `try-catch`** | Handles Slack notification errors |
| **Email `try-catch`** | Handles email notification errors separately |
| **`post { always }`** | Attempts cleanup after pipeline execution |
| **`cleanWs()`** | Removes workspace files and directories |

## 8. Conclusion

This Jenkins Shared Library standardizes Golang dependency scanning by automating dependency preparation, vulnerability detection with `govulncheck`, report generation, and artifact archiving. It also supports Slack and email notifications, error handling, and workspace cleanup.

## 9. Contact Information

| Property | Details |
|---|---|
| **Name** | Ritu |
| **Email** | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

## 10. References

- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [Go Documentation](https://go.dev/doc/)
- [Go Modules Reference](https://go.dev/ref/mod)
- [govulncheck Documentation](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)
- [Go Vulnerability Database](https://vuln.go.dev/)
