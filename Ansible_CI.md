# Jenkins Shared Library for Golang CI Pipeline

<p align="center">
  <img width="333" height="155" alt="Golang CI Pipeline" src="https://github.com/user-attachments/assets/be9907e0-f42a-4213-af2f-15eb5e0aacd7" />
</p>

---

# Document Information

| **Author** | **Created On** | **Version** | **L0 Reviewer** | **L1 Reviewer** | **L2 Reviewer** |
|---|---|---|---|---|---|
| Ritu | 27/10/2026 | 1.0 | Liyakhat/Anirudh | Aman Raj | Sandeep Rawat/Ravindra |

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

# 1. Introduction

This project implements a Jenkins Shared Library to automate the Continuous Integration (CI) process for Go-based applications.

The library uses Go tools to download project dependencies, run tests, generate a code coverage report, compile the application, and archive generated artifacts in Jenkins. It also sends build-status notifications through Slack and email.

---

# 2. Library Type

| **Property** | Details |
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

---

# 3. Library Workflow

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
Download Go Dependencies
      |
      v
Run Go Tests and Generate Coverage
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
Send Slack Notification
      |
      v
Send Email Notification
      |
      v
Clean Jenkins Workspace
      |
      v
End Pipeline
```

The Jenkins pipeline checks out the Employee API repository and invokes the reusable `goCI()` function. The library prepares dependencies, runs tests, generates coverage, compiles the application, and archives artifacts. Jenkins then sends notifications according to the pipeline result and cleans the workspace.

---

# 4. Prerequisites and Dependencies

## 4.1 Jenkins Requirements

| **Requirement** | Purpose |
|---|---|
| **Jenkins** | Executes the CI pipeline |
| **Pipeline Plugin** | Supports pipeline execution |
| **Git Plugin** | Checks out the application repository |
| **Configured Shared Library** | Provides reusable CI and notification functions |
| **Slack Notification Plugin** | Provides the `slackSend()` step |
| **Email Extension Plugin** | Provides the `emailext()` step |
| **Workspace Cleanup Plugin** | Provides the `cleanWs()` step |
| **Artifact Archiving Support** | Stores the coverage report and compiled binary |

## 4.2 Build Agent Requirements

| **Dependency** | Purpose |
|---|---|
| **Go** | Downloads dependencies, runs tests, and compiles the application |
| **Git** | Retrieves application source code |
| **Network Connectivity** | Allows access to GitHub and Go module repositories |

The Jenkins agent must have Go installed and available in its system `PATH`. The shared library uses the system-default Go installation.

---

# 5. Implementation Details

## 5.1 Configure Shared Library

The Jenkinsfile loads the shared library from GitHub.

```groovy
library identifier: 'Shared_Library@SCRUM-291-RITU',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        remote: 'https://github.com/SnaatakAllStars/Shared_Library.git',
        credentialsId: 'Github-cred'
    ])
```

**Purpose:**

- Retrieves the shared library from GitHub.
- Uses the `SCRUM-291-RITU` branch.
- Uses `Github-cred` for repository authentication.
- Makes reusable CI and notification functions available.

## 5.2 Run Golang CI Process

The Jenkins pipeline invokes the reusable `goCI()` function.

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

**Purpose:**

- Calls `goCI.groovy`.
- Passes the application directory through `appDir`.
- Enables testing with `runTests: true`.
- Executes dependency, testing, coverage, and compilation steps.

## 5.3 Check Go Version

The shared library checks the Go version installed on the Jenkins agent.

```bash
set -eu
echo "[INFO] Go version:"
go version
```

**Purpose:**

- Verifies that Go is available.
- Displays the installed Go version in the Jenkins console.
- Uses the system-default Go installation.

## 5.4 Download Project Dependencies

The shared library prepares dependencies using Go Modules.

```bash
set -eu
go mod tidy
go mod download
```

**Purpose:**

- `go mod tidy` updates module definitions and dependency checksums as needed.
- `go mod download` downloads required Go modules.
- Prepares dependencies for testing and compilation.

**Note:** `go mod tidy` can modify `go.mod` and `go.sum` in the workspace.

## 5.5 Run Go Tests and Generate Coverage

When `runTests` is enabled, the shared library executes:

```bash
set -eu
mkdir -p reports
go test -v -coverprofile=reports/coverage.out ./... || true
```

**Purpose:**

- Creates the `reports` directory.
- Runs tests across Go packages using `./...`.
- Uses `-v` to display detailed test output.
- Generates the coverage profile at `reports/coverage.out`.

The `-coverprofile` option records coverage information for tested packages.

**Important:** `|| true` suppresses test failures, so the pipeline may continue to compilation even if tests fail.

## 5.6 Archive Coverage Report

The shared library archives the coverage report in Jenkins.

```groovy
archiveArtifacts(
    artifacts: 'reports/coverage.out',
    allowEmptyArchive: true
)
```

**Purpose:**

- Archives `reports/coverage.out`.
- Makes the coverage profile available from the Jenkins build.
- Allows the team to review generated coverage data.

## 5.8 Compile Go Application

The shared library compiles the Go application.

```bash
set -eu
go build -o app-binary main.go
```

**Purpose:**

- Compiles the application using Go.
- Uses `main.go` as the input file.
- Generates a binary named `app-binary`.
- Stops the shell step if compilation fails.

This command assumes `main.go` exists in the application directory and can be built independently.

## 5.7 Archive Compiled Binary

After compilation, the library archives the generated binary.

```groovy
archiveArtifacts(
    artifacts: 'app-binary',
    allowEmptyArchive: true
)
```

**Purpose:**

- Archives the compiled `app-binary`.
- Makes the binary available through Jenkins build artifacts.
- Allows users to retrieve the compiled output from the build page.

---

# 6. Notifications

The `notifyStatus.groovy` function sends build-status notifications through Slack and email.

The Jenkinsfile calls this function from its `post` section after the pipeline completes.

**Slack message includes:**

- Build status
- Jenkins job name
- Build number
- Build URL

**Email includes:**

- Build status in the subject
- Pipeline stage name
- Jenkins job name and build number
- Jenkins console URL
- Jenkins artifact URL

---

# 7. Error Handling and Cleanup

The Jenkins pipeline and shared library use shell error handling, notification exception handling, artifact archiving, and workspace cleanup.

| **Mechanism** | Purpose |
|---|---|
| **`set -eu`** | Stops shell execution when a command fails or an unset variable is used |
| **`|| true`** | Allows execution to continue even if the Go test command fails |
| **`archiveArtifacts()`** | Archives the coverage report and compiled binary |
| **Slack `try-catch`** | Handles Slack notification errors separately |
| **Email `try-catch`** | Handles email notification errors separately |
| **`post { always }`** | Attempts workspace cleanup after pipeline execution |
| **`post { success }`** | Sends success notifications |
| **`post { failure }`** | Sends failure notifications |
| **`cleanWs()`** | Removes workspace files and directories |

---

# 8. Conclusion

This document describes the Jenkins Shared Library implemented for Go application CI automation.

The library automates dependency preparation, Go testing, coverage report generation, application compilation, and artifact archiving. It also provides reusable Slack and email notifications, error handling, and workspace cleanup.

This approach reduces duplicated pipeline code and provides a consistent CI workflow for Go-based applications.

---

# 9. Contact Information

| **Name** | **Email** |
|---|---|
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

---

# 10. References

- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [Go Documentation](https://go.dev/doc/)
- [Go Modules Reference](https://go.dev/ref/mod)
- [Go Testing Package](https://pkg.go.dev/testing)
- [Go Build Command](https://pkg.go.dev/cmd/go#hdr-Compile_packages_and_dependencies)
- [Slack Notification Plugin](https://plugins.jenkins.io/slack/)
