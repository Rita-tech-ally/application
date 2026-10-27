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

This project implements a Jenkins Shared Library to automate Continuous Integration (CI) for React applications.

The library uses Node.js and npm to install dependencies, execute tests, generate reports, build the application, and archive artifacts. It also publishes reports in Jenkins and sends Slack and email notifications.

Reusable functions reduce duplicate pipeline code and simplify maintenance.

---

# 2. Library Type

| Property | Details |
|---|---|
| Library Type | Jenkins Shared Library |
| Language | Groovy |
| Application | React |
| Runtime | Node.js 18 |
| Package Manager | npm |
| Testing | Project-configured test runner with Jest options |
| Test Report | JUnit XML |
| Coverage Report | HTML |
| CI Platform | Jenkins |
| Source Control | GitHub |
| Notifications | Slack and Email |
| Artifacts | Jenkins Archived Artifacts |

---

# 3. Library Workflow

```text
Start Pipeline
      |
      v
Checkout Frontend Repository
      |
      v
Load Shared Library and Call reactCI()
      |
      v
Configure Node.js
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

The Jenkinsfile checks out the application repository and invokes `reactCI()`. The library handles dependency installation, testing, report publishing, building, and artifact archiving. Notifications and cleanup run through Jenkins pipeline post-actions.

---

# 4. Prerequisites and Dependencies

## 4.1 Jenkins Requirements

| Requirement | Purpose |
|---|---|
| Jenkins | Executes the pipeline |
| Pipeline Plugin | Supports pipeline execution |
| Git Plugin | Checks out source code |
| NodeJS Plugin | Configures Node.js |
| JUnit Plugin | Publishes test results |
| HTML Publisher Plugin | Publishes coverage reports |
| Slack Plugin | Sends Slack notifications |
| Email Extension Plugin | Sends email notifications |
| Workspace Cleanup Plugin | Cleans workspace files |

## 4.2 Build Agent Requirements

- Node.js 18 and npm must be configured in Jenkins as `NodeJS_18`.
- Git must be installed on the build agent.
- Network connectivity to GitHub and npm registries is required.
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

- Retrieves the shared library from GitHub.
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

The repository URL, branch, directory, and credentials are supplied through environment variables.

## 5.3 Run React CI Process

The Jenkinsfile invokes the reusable `reactCI()` function.

```groovy
reactCI(
    appDir: env.APP_REPO_NAME,
    nodeTool: 'NodeJS_18',
    runTests: true
)
```

| Parameter | Purpose |
|---|---|
| `appDir` | Application directory |
| `nodeTool` | Configured Node.js installation |
| `runTests` | Enables test execution |

## 5.4 Configure Node.js

The library activates Node.js and verifies the installed versions.

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

This makes Node.js and npm available for subsequent CI steps.

## 5.5 Install Dependencies

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
- Uses `npm ci` when `package-lock.json` exists.
- Uses `npm install` when the lockfile is absent.
- Removes the temporary `.npmrc` after successful installation.

**Note:** `ignore-scripts=true` disables dependency lifecycle scripts and may affect packages that require installation scripts. Because the shell uses `set -e`, `.npmrc` may remain if installation fails.

## 5.6 Run React Tests

When `runTests` is enabled, the library configures JUnit output and executes tests.

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

This command runs tests, requests HTML and text coverage reports, and configures the JUnit XML output location.

**Important:** `|| true` suppresses the test command's failure status. Consequently, failing tests may not fail the pipeline.

## 5.7 Publish Test and Coverage Reports

The JUnit plugin publishes test results.

```groovy
junit(
    testResults: 'test-results/junit.xml',
    allowEmptyResults: true
)
```

The HTML Publisher plugin publishes coverage results.

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

These reports make test results and coverage information accessible from the Jenkins build page. The configured paths must match the generated files.

## 5.8 Build React Application

The library runs the project's build script.

```bash
set -eu
export CI=false
export NODE_OPTIONS=--openssl-legacy-provider
npm run build
```

The build command generates the production application. This pipeline expects the output in the `build` directory.

## 5.9 Archive Build Artifacts

The library archives the build and coverage files.

```groovy
archiveArtifacts(
    artifacts: 'build/**, coverage/**',
    allowEmptyArchive: true
)
```

Archived files are accessible from the Jenkins build page. JUnit results are published separately through the JUnit plugin.

---

# 6. Notifications

The `notifyStatus.groovy` function sends build notifications through Slack and email.

## 6.1 Slack Notification

The `slackSend()` step sends a summary containing:

- Build status.
- Jenkins job name and build number.
- Jenkins console URL.
- Relevant pipeline information.

Success and failure notifications use different status colors.

## 6.2 Email Notification

The `emailext()` step sends an email containing:

- Build status.
- Jenkins job name and build number.
- Jenkins console URL.
- Relevant build information.

The Jenkinsfile calls the notification function from its `post` section for successful and failed builds.

---

# 7. Error Handling and Cleanup

| Mechanism | Purpose |
|---|---|
| `set -eu` | Stops shell execution on command failures or unset variables |
| `|| true` | Suppresses the test command's failure status |
| `allowEmptyResults` | Allows missing JUnit results |
| `allowMissing` | Allows missing HTML reports |
| `allowEmptyArchive` | Allows an empty artifact archive |
| `post { success }` | Handles successful builds |
| `post { failure }` | Handles failed builds |
| `post { always }` | Runs actions after pipeline completion |
| `cleanWs()` | Removes workspace files and directories |

Cleanup should run in the `always` post-condition so the workspace is cleaned after successful or failed builds. Notification errors should be handled separately where necessary.

---

# 8. Conclusion

The Jenkins Shared Library automates React CI tasks, including dependency installation, test execution, report publishing, application builds, and artifact archiving.

Reusable CI and notification functions improve consistency and simplify pipeline maintenance across frontend projects.

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
