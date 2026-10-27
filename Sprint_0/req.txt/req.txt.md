# Jenkins Shared Library for Golang CI Pipeline

<p align="center">
  <img width="333" height="155" alt="Golang CI Pipeline" src="https://github.com/user-attachments/assets/be9907e0-f42a-4213-af2f-15eb5e0aacd7" />
</p>

## Document Information

| Property | Details |
|---|---|
| **Author** | Ritu |
| **Created On** | 27/10/2026 |
| **Version** | 1.0 |
| **L0 Reviewer** | Liyakhat / Anirudh |
| **L1 Reviewer** | Aman Raj |
| **L2 Reviewer** | Sandeep Rawat / Ravindra |

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

## 1. Introduction

This Jenkins Shared Library automates the Continuous Integration (CI) process for Go-based applications. It downloads dependencies, runs tests, generates coverage reports, compiles the application, archives artifacts, and sends build notifications.

## 2. Library Type

| Property | Details |
|---|---|
| **Library Type** | Jenkins Shared Library |
| **Language** | Groovy |
| **Application Type** | Go-based applications |
| **CI Process** | Continuous Integration |
| **Build Tool** | Go |
| **Dependency Management** | Go Modules |
| **Testing** | Go Test |
| **Code Coverage** | Go Coverage |
| **CI Platform** | Jenkins |
| **Source Control** | GitHub |
| **Notifications** | Slack and Email |
| **Artifact Management** | Jenkins Archived Artifacts |

## 3. Library Workflow

```text
Start Pipeline
      |
      v
Checkout Employee API Repository
      |
      v
Call goCI()
      |
      v
Check Go Version
      |
      v
Download Dependencies
      |
      v
Run Tests and Generate Coverage
      |
      v
Archive Coverage Report
      |
      v
Compile Go Application
      |
      v
Archive Compiled Binary
      |
      v
Determine Pipeline Status
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
| **Jenkins** | Runs the CI pipeline |
| **Pipeline Plugin** | Supports pipeline execution |
| **Git Plugin** | Checks out source code |
| **Configured Shared Library** | Loads reusable CI functions |
| **Slack Notification Plugin** | Sends Slack notifications |
| **Email Extension Plugin** | Sends email notifications |
| **Workspace Cleanup Plugin** | Removes workspace files |
| **Artifact Archiving Support** | Stores build artifacts |

### 4.2 Build Agent Requirements

| Dependency | Purpose |
|---|---|
| **Go** | Builds and tests the application |
| **Git** | Retrieves source code |
| **Network Connectivity** | Downloads dependencies |

Go must be installed and available in the agent's `PATH`. The pipeline uses the system-default Go installation.

## 5. Implementation Details

### 5.1 Load the Shared Library

```groovy
library identifier: 'Shared_Library@SCRUM-291-RITU',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        remote: 'https://github.com/SnaatakAllStars/Shared_Library.git',
        credentialsId: 'Github-cred'
    ])
```

This loads the shared library from GitHub using the configured Jenkins credentials.

### 5.2 Execute Golang CI

```groovy
stage('Golang CI Process') {
    steps {
        goCI(
            appDir: env.APP_REPO_NAME,
            runTests: true
        )
    }
}
```

The `goCI()` function executes the reusable Go CI steps for the application directory.

### 5.3 Check Go Version

```bash
set -eu
echo "[INFO] Go version:"
go version
```

This verifies that Go is installed and available.

### 5.4 Download Dependencies

```bash
set -eu
go mod tidy
go mod download
```

- `go mod tidy` updates `go.mod` and `go.sum` as needed.
- `go mod download` downloads the required Go modules.

**Note:** `go mod tidy` may modify dependency files.

### 5.5 Run Tests and Generate Coverage

```bash
set -eu
mkdir -p reports
go test -v -coverprofile=reports/coverage.out ./... || true
```

- `go test` runs tests across Go packages.
- `-coverprofile` generates a coverage data file.
- `|| true` prevents test failures from stopping this command. Therefore, failed tests may not fail the pipeline at this step.

### 5.6 Archive Coverage Report

```groovy
archiveArtifacts(
    artifacts: 'reports/coverage.out',
    allowEmptyArchive: true
)
```

This archives the coverage report in Jenkins. `allowEmptyArchive: true` allows execution to continue if the report is missing.

### 5.7 Compile the Go Application

```bash
set -eu
go build -o app-binary main.go
```

This compiles `main.go` into an executable named `app-binary`. It assumes `main.go` can be built independently.

### 5.8 Archive the Compiled Binary

```groovy
archiveArtifacts(
    artifacts: 'app-binary',
    allowEmptyArchive: true
)
```

This stores the compiled binary as a Jenkins build artifact.

## 6. Notifications

The shared library uses `notifyStatus.groovy` to send build notifications.

| Notification | Details |
|---|---|
| **Slack** | Build status, job name, build number, and build URL |
| **Email** | Build status, pipeline stage, job/build number, console URL, and artifact URL |

Notifications help the team track pipeline results and investigate failures.

## 7. Error Handling and Cleanup

| Mechanism | Purpose |
|---|---|
| **`set -eu`** | Stops shell execution on errors and unset variables |
| **`|| true`** | Prevents the test command's failure from stopping execution |
| **`archiveArtifacts()`** | Stores build outputs in Jenkins |
| **Slack `try-catch`** | Handles Slack notification errors |
| **Email `try-catch`** | Handles email notification errors |
| **`post { always }`** | Executes cleanup regardless of the build result |
| **`post { success }`** | Handles successful build actions |
| **`post { failure }`** | Handles failed build actions |
| **`cleanWs()`** | Cleans the Jenkins workspace |

**Important:** Because `|| true` suppresses the test command's exit status, tests may fail while the pipeline continues. Remove it or explicitly capture test results if failed tests must fail the build.

## 8. Conclusion

The Jenkins Shared Library standardizes the Go CI process by reusing common pipeline steps for dependency management, testing, coverage generation, compilation, artifact archiving, notifications, and cleanup.

## 9. Contact Information

| Property | Details |
|---|---|
| **Name** | Ritu |
| **Email** | ritu.dogra.snaatak@mygurukulam.co |

## 10. References

- [Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [Go Documentation](https://go.dev/doc/)
- [Go Modules](https://go.dev/ref/mod)
- [Go Testing](https://pkg.go.dev/testing)
- [Go Build Command](https://pkg.go.dev/cmd/go#hdr-Compile_packages_and_dependencies)
- [Jenkins Slack Plugin](https://plugins.jenkins.io/slack/)
