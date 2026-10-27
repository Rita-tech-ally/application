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

# 1. Introduction

This project implements a Jenkins Shared Library to automate the Continuous Integration (CI) process for React-based applications.

The library uses Node.js and npm to install dependencies, execute tests, generate test and coverage reports, and build the React application. It also archives build artifacts, publishes reports in Jenkins, and sends notifications through Slack and email.

The shared library provides reusable CI functions, reducing duplicate pipeline code and simplifying maintenance.

---

# 2. Library Type

| Property | Details |
|---|---|
| Library Type | Jenkins Shared Library |
| Language | Groovy |
| Application Type | React |
| CI Process | Continuous Integration |
| Runtime | Node.js 18 |
| Package Manager | npm |
| Testing | Project-configured test runner with Jest options |
| Test Report | JUnit XML |
| Coverage Report | HTML |
| CI Platform | Jenkins |
| Source Control | GitHub |
| Notifications | Slack and Email |
| Artifact Management | Jenkins Archived Artifacts |

---

# 3. Library Workflow

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
Publish Test and Coverage Reports
      |
      v
Build React Application
      |
      v
Archive Build Artifacts
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

The Jenkinsfile checks out the Frontend repository and calls `reactCI()`. The library handles dependency installation, testing, report publishing, building, and artifact archiving. Jenkins then sends notifications according to the pipeline result and cleans the workspace.

---

# 4. Prerequisites and Dependencies

## 4.1 Jenkins Requirements

| Requirement | Purpose |
|---|---|
| Jenkins | Executes the CI pipeline |
| Pipeline Plugin | Supports pipeline execution |
| Git Plugin | Checks out application code |
| NodeJS Plugin | Provides the Node.js installation |
| JUnit Plugin | Publishes test reports |
| HTML Publisher Plugin | Publishes coverage reports |
| Slack Plugin | Sends Slack notifications |
| Email Extension Plugin | Sends email notifications |
| Workspace Cleanup Plugin | Cleans the workspace |

## 4.2 Build Agent Requirements

- Node.js 18 and npm must be available through the Jenkins installation named `NodeJS_18`.
- Git must be installed.
- The agent requires network connectivity to GitHub and npm registries.
- The agent must have permission to generate reports and build artifacts.
- The shared library and GitHub credentials must be configured in Jenkins.

---

# 5. Implementation Details

## 5.1 Configure Shared Library

The Jenkinsfile loads the shared library from GitHub.

```groovy
library identifier: 'allstars-shared-library@SCRUM-258-RITU',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        remote: 'https://github.com/SnaatakAllStars/Shared_Library.git',
        credentialsId: 'Github-cred'
    ])
```

**Purpose:**

- Retrieves the library from GitHub.
- Uses the `SCRUM-258-RITU` branch.
- Uses `Github-cred` for authentication.
- Makes reusable CI and notification functions available.

## 5.2 Checkout Application Repository

The pipeline checks out the Frontend repository into the configured application directory.

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

This stage retrieves the application source code from the configured GitHub repository and branch.

## 5.3 Run React CI Process

The pipeline invokes the reusable `reactCI()` function.

```groovy
reactCI(
    appDir: env.APP_REPO_NAME,
    nodeTool: 'NodeJS_18',
    runTests: true
)
```

| Parameter | Purpose |
|---|---|
| `appDir` | Specifies the application directory |
| `nodeTool` | Selects the configured Node.js installation |
| `runTests` | Enables test execution |

## 5.4 Configure Node.js Environment

The library activates the configured Node.js installation and checks the installed versions.

```groovy
dir(appDir) {
    nodejs(nodeTool) {
        sh '''
            set -eu
            node --version
            npm --version
        '''
    }
}
```

This makes Node.js and npm available for subsequent CI operations.

## 5.5 Install Project Dependencies

The library configures npm and installs application dependencies.

```bash
export DISABLE_OPENCOLLECTIVE=true
export ADBLOCK=true
export SUPPRESS_SUPPORT=true

echo "ignore-scripts=true" > .npmrc
echo "fund=false" >> .npmrc

if [ -f package-lock.json ]; then
    npm ci --loglevel=error
else
    npm install --loglevel=error
fi

rm .npmrc
```

**Purpose:**

- Reduces selected npm output.
- Creates a temporary `.npmrc` configuration.
- Uses `npm ci` when `package-lock.json` exists.
- Uses `npm install` when the lockfile is absent.
- Removes the temporary configuration after successful installation.

**Note:** `ignore-scripts=true` disables dependency lifecycle scripts, which may affect packages requiring installation scripts.

## 5.6 Run React Tests

When `runTests` is enabled, the library configures JUnit output and executes the project test command.

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

This step runs tests, requests HTML and text coverage reports, and configures the JUnit XML report location.

**Important:** `|| true` suppresses the test command's failure status. Therefore, failing tests may not cause the pipeline to fail.

## 5.7 Publish Test and Coverage Reports

The JUnit plugin publishes test results.

```groovy
junit(
    testResults: 'test-results/junit.xml',
    allowEmptyResults: true
)
```

The HTML Publisher plugin publishes the coverage report.

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

These reports make test results and coverage information accessible from the Jenkins build page. The configured report paths must match the generated output.

## 5.8 Build React Application

The library executes the application build command.

```bash
set -eu
export CI=false
export NODE_OPTIONS=--openssl-legacy-provider
npm run build
```

This runs the project's build script. The pipeline expects the generated build output in the `build` directory.

## 5.9 Archive Build Artifacts

The library archives the generated build and coverage files.

```groovy
archiveArtifacts(
    artifacts: 'build/**, coverage/**',
    allowEmptyArchive: true
)
```

Archived artifacts are available from the Jenkins build page. The test-results directory is published separately through the JUnit plugin.

---

# 6. Notifications

The `notifyStatus.groovy` function sends build notifications through Slack and email.

## 6.1 Slack Notification

The `slackSend()` step sends a build summary containing:

- Build status.
- Jenkins job name and build number.
- Jenkins console URL.
- Relevant pipeline information.

Success and failure notifications use different status colors.

## 6.2 Email Notification

The `emailext()` step sends an email containing:

- Build status and stage name.
- Jenkins job name and build number.
- Jenkins console URL.

The Jenkinsfile calls the notification function from its `post` section for successful and failed builds.

---

# 7. Error Handling and Cleanup

| Mechanism | Purpose |
|---|---|
| `set -eu` | Stops shell execution on command failures or unset variables |
| `|| true` | Prevents test command failures from propagating |
| `allowEmptyResults` | Allows missing JUnit results |
| `allowMissing` | Allows missing HTML reports |
| `allowEmptyArchive` | Allows empty artifact archives |
| `post { success }` | Sends success notifications |
| `post { failure }` | Sends failure notifications |
| `post { always }` | Runs cleanup after pipeline execution |
| `cleanWs()` | Removes workspace files and directories |

Notification failures should be handled separately so that Slack or email delivery issues do not unnecessarily prevent workspace cleanup.

---

# 8. Conclusion

The Jenkins Shared Library automates React CI operations, including dependency installation, test execution, report publishing, application builds, and artifact archiving.

Reusable functions for CI execution and notifications improve consistency and simplify pipeline maintenance across frontend projects.

---

# 9. Contact Information

| Name | Email |
|---|---|
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

---

# 10. References

- [Jenkins Pipeline Documentation](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [NodeJS Plugin](https://plugins.jenkins.io/nodejs/)
- [npm Documentation](https://docs.npmjs.com/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [JUnit Plugin](https://plugins.jenkins.io/junit/)
- [HTML Publisher Plugin](https://plugins.jenkins.io/htmlpublisher/)
- [Slack Plugin](https://plugins.jenkins.io/slack/)
- [Email Extension Plugin](https://plugins.jenkins.io/email-ext/)
- [Workspace Cleanup Plugin](https://plugins.jenkins.io/ws-cleanup/)
