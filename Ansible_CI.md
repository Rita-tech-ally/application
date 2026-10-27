# Jenkins Shared Library for React CI Pipeline

<p align="center">
  <img width="333" height="155" alt="React CI Pipeline" src="https://github.com/user-attachments/assets/be9907e0-f42a-4213-af2f-15eb5e0aacd7" />
</p>

---

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
|---|---|---|---|---|---|
| Ritu | 25/10/2026 | 1.0 | Liyakhat/Anirudh | Aman Raj | Sandeep Rawat/Ravindra |

---

## Table of Contents

1. Introduction
2. Objectives
3. Library Type
4. Repository Structure
5. Library Workflow
6. Prerequisites and Dependencies
7. Shared Library Configuration
8. Implementation Details
9. Notifications
10. Error Handling and Cleanup
11. Validation
12. Conclusion
13. Contact Information
14. References

---

## 1. Introduction

This project implements a Jenkins Shared Library to automate the Continuous Integration (CI) process for React-based applications.

The library uses Node.js and npm to install project dependencies, execute tests, generate test and coverage reports, and build the React application.

It also publishes test reports in Jenkins, archives build artifacts, sends build-status notifications through Slack and email, and cleans the Jenkins workspace after execution.

The shared library makes the CI workflow reusable and easier to maintain across frontend projects.

## 2. Objectives

- Automate the React application CI process.
- Reuse pipeline logic through Jenkins Shared Library.
- Configure Node.js and npm for application builds.
- Install project dependencies.
- Execute tests and generate coverage reports.
- Publish JUnit XML and HTML coverage reports.
- Build the React application.
- Archive generated build artifacts.
- Send Slack and email notifications.
- Clean the Jenkins workspace after execution.

---

## 3. Library Type

| Property | Details |
|---|---|
| Library Type | Jenkins Shared Library |
| Language | Groovy |
| Application Type | React |
| CI Process | Continuous Integration |
| Runtime | Node.js 18 |
| Package Manager | npm |
| Testing Framework | Project-configured test runner |
| Test Report | JUnit XML |
| Coverage Report | HTML |
| Source Control | GitHub |
| CI Platform | Jenkins |
| Notifications | Slack and Email |
| Artifact Management | Jenkins Archived Artifacts |

---

## 4. Repository Structure

```text
Shared_Library/
├── vars/
│   ├── reactCI.groovy
│   └── notifyStatus.groovy
└── README.md
```

| File | Purpose |
|---|---|
| `reactCI.groovy` | Automates dependency installation, tests, reporting, and builds. |
| `notifyStatus.groovy` | Sends Slack and email notifications. |
| `README.md` | Documents library configuration and usage. |

---

## 5. Library Workflow

```text
Start Pipeline
      |
      v
Checkout Frontend Repository
      |
      v
Call reactCI()
      |
      v
Configure Node.js Environment
      |
      v
Install npm Dependencies
      |
      v
Run React Tests
      |
      v
Generate Test and Coverage Reports
      |
      v
Publish Reports in Jenkins
      |
      v
Build React Application
      |
      v
Archive Build Artifacts
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

The pipeline checks out the frontend source code and invokes the reusable `reactCI()` function.

The library performs the configured CI operations and makes generated reports and artifacts available through Jenkins.

---

## 6. Prerequisites and Dependencies

### 6.1 Jenkins Requirements

| Requirement | Purpose |
|---|---|
| Jenkins | Executes the CI pipeline. |
| Pipeline Plugin | Supports pipeline execution. |
| Git Plugin | Checks out application code. |
| NodeJS Plugin | Provides the configured Node.js installation. |
| JUnit Plugin | Publishes test results. |
| HTML Publisher Plugin | Publishes HTML coverage reports. |
| Slack Plugin | Sends Slack notifications. |
| Email Extension Plugin | Sends email notifications. |
| Workspace Cleanup Plugin | Cleans the workspace. |

### 6.2 Build Agent Requirements

- Node.js 18 must be configured in Jenkins.
- npm must be available through the Node.js installation.
- Git must be installed.
- The agent must have network connectivity to GitHub and npm registries.
- The agent must have permission to create reports and build artifacts.

The configured Node.js installation name is `NodeJS_18`.

---

## 7. Shared Library Configuration

The Jenkinsfile loads the shared library from GitHub.

```groovy
library identifier: 'allstars-shared-library@SCRUM-258-RITU',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        remote: 'https://github.com/SnaatakAllStars/Shared_Library.git',
        credentialsId: 'Github-cred'
    ])
```

**Configuration details:**

- Repository: `SnaatakAllStars/Shared_Library`
- Branch: `SCRUM-258-RITU`
- Credentials ID: `Github-cred`
- Retrieval method: Modern SCM

The library makes reusable React CI and notification functions available to the pipeline.

---

## 8. Implementation Details

### 8.1 Checkout Application Repository

The pipeline checks out the frontend repository into the configured application directory.

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

**Purpose:**

- Retrieves source code from GitHub.
- Checks out the configured branch.
- Stores the application in the workspace directory.

### 8.2 Invoke React CI Function

```groovy
reactCI(
    appDir: env.APP_REPO_NAME,
    nodeTool: 'NodeJS_18',
    runTests: true
)
```

The function accepts three parameters:

| Parameter | Purpose |
|---|---|
| `appDir` | Specifies the application directory. |
| `nodeTool` | Selects the configured Node.js installation. |
| `runTests` | Enables test execution when set to `true`. |

### 8.3 Configure Node.js

The library uses the Jenkins NodeJS installation.

```groovy
nodejs(nodeTool) {
    sh '''
        node --version
        npm --version
    '''
}
```

This makes Node.js and npm available to the subsequent CI commands.

### 8.4 Install Dependencies

The library configures npm and installs the application dependencies.

```bash
export DISABLE_OPENCOLLECTIVE=true
export ADBLOCK=true
export SUPPRESS_SUPPORT=true

if [ -f package-lock.json ]; then
    npm ci --loglevel=error
else
    npm install --loglevel=error
fi
```

- `npm ci` uses the existing lockfile for a reproducible installation.
- `npm install` is used when no lockfile exists.
- npm output is reduced through the configured environment variables and log level.

The implementation also creates a temporary `.npmrc` with `ignore-scripts=true` and `fund=false`, then removes it after successful installation.

**Note:** Disabling lifecycle scripts can affect packages that require installation scripts.

### 8.5 Execute React Tests

When `runTests` is enabled, the library configures JUnit reporting and executes the project's test command.

```bash
export CI=true
export JEST_JUNIT_OUTPUT_DIR="./test-results"
export JEST_JUNIT_OUTPUT_NAME="junit.xml"

npm test -- \
    --coverage \
    --coverageReporters="html" \
    --coverageReporters="text" \
    --reporters=default \
    --reporters=jest-junit \
    --passWithNoTests || true
```

This step:

- Runs the configured test script.
- Requests HTML and text coverage output.
- Configures the JUnit report location.
- Allows execution when no tests are present.

**Important:** `|| true` suppresses the command's failure status. Consequently, failing tests may not fail the pipeline.

### 8.6 Publish JUnit Test Results

```groovy
junit(
    testResults: 'test-results/junit.xml',
    allowEmptyResults: true
)
```

The JUnit plugin publishes test results on the Jenkins build page.

The report helps review test execution results after the pipeline completes.

### 8.7 Publish HTML Coverage Report

```groovy
publishHTML(target: [
    allowMissing: true,
    alwaysLinkToLastBuild: true,
    keepAll: true,
    reportDir: 'coverage/lcov-report',
    reportFiles: 'index.html',
    reportName: 'HTML Coverage Report'
])
```

The HTML Publisher plugin makes the coverage report accessible through Jenkins.

The report directory must match the location generated by the application's coverage tooling.

### 8.8 Build React Application

```bash
set -eu
export CI=false
export NODE_OPTIONS=--openssl-legacy-provider
npm run build
```

This step executes the application's build script and generates production build output.

The implementation expects the generated directory to be named `build`.

### 8.9 Archive Build Artifacts

```groovy
archiveArtifacts(
    artifacts: 'build/**, coverage/**',
    allowEmptyArchive: true
)
```

The library archives files from the build and coverage directories.

Archived artifacts can be accessed from the Jenkins build page.

---

## 9. Notifications

The `notifyStatus.groovy` function sends notifications based on the pipeline result.

### 9.1 Slack Notification

The library uses `slackSend()` to send build information to Slack.

The notification can include:

- Build status.
- Jenkins job name.
- Build number.
- Jenkins console URL.

### 9.2 Email Notification

The library uses `emailext()` to send a build summary.

The email includes the build status, job details, build number, and Jenkins console link.

The Jenkins `post` section invokes notifications for successful and failed executions.

---

## 10. Error Handling and Cleanup

| Mechanism | Purpose |
|---|---|
| `set -eu` | Stops shell execution on command failures or unset variables. |
| `|| true` | Prevents the test command from failing the shell step. |
| `allowEmptyResults` | Allows missing JUnit results. |
| `allowMissing` | Allows missing HTML reports. |
| `allowEmptyArchive` | Allows artifact archiving with no matching files. |
| `post { success }` | Sends success notifications. |
| `post { failure }` | Sends failure notifications. |
| `post { always }` | Executes cleanup regardless of the build result. |
| `cleanWs()` | Removes workspace files. |

Slack and email notification errors should be handled independently so that notification failures do not unnecessarily interrupt cleanup.

---

## 11. Validation

|
