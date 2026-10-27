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

This Jenkins Shared Library automates CI for React applications using Node.js and npm. It installs dependencies, runs tests, generates reports, builds the application, archives artifacts, and sends Slack and email notifications.

# 2. Library Type

| Property | Details |
|---|---|
| Type | Jenkins Shared Library |
| Language | Groovy |
| Application | React |
| Runtime | Node.js 18 |
| Package Manager | npm |
| Reports | JUnit XML and HTML Coverage |
| Source Control | GitHub |
| Notifications | Slack and Email |

# 3. Library Workflow

```text
Start Pipeline
      |
      v
Checkout Frontend Repository
      |
      v
Load Shared Library
      |
      v
Install Dependencies
      |
      v
Run Tests and Publish Reports
      |
      v
Build React Application
      |
      v
Archive Artifacts
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

# 4. Prerequisites and Dependencies

## 4.1 Jenkins Requirements

| Requirement | Purpose |
|---|---|
| Jenkins Pipeline Plugin | Pipeline execution |
| Git Plugin | Repository checkout |
| NodeJS Plugin | Node.js setup |
| JUnit Plugin | Test reports |
| HTML Publisher Plugin | Coverage reports |
| Slack Plugin | Slack notifications |
| Email Extension Plugin | Email notifications |
| Workspace Cleanup Plugin | Workspace cleanup |

## 4.2 Build Agent Requirements

- Node.js 18 and npm configured as `NodeJS_18`.
- Git installed on the agent.
- Network access to GitHub and npm.
- Shared Library and GitHub credentials configured in Jenkins.

# 5. Implementation Details

## 5.1 Load Shared Library

```groovy
library identifier: 'allstars-shared-library@SCRUM-258-RITU',
    retriever: modernSCM([
        $class: 'GitSCMSource',
        remote: 'https://github.com/SnaatakAllStars/Shared_Library.git',
        credentialsId: 'Github-cred'
    ])
```

This loads reusable pipeline functions from the configured GitHub branch.

## 5.2 Checkout Application Repository

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

The repository URL, branch, and credentials are provided through environment variables.

## 5.3 Execute React CI

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
| `nodeTool` | Node.js installation |
| `runTests` | Enables testing |

## 5.4 Configure Node.js

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

This activates Node.js and verifies Node.js and npm versions.

## 5.5 Install Dependencies

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

- `npm ci` installs dependencies from the lockfile.
- `npm install` is used when the lockfile is absent.
- `ignore-scripts=true` disables dependency lifecycle scripts and may affect packages that require them.

## 5.6 Run Tests

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

This runs tests and requests JUnit XML and coverage reports.

**Note:** `|| true` suppresses test failures, so failing tests may not fail the pipeline.

## 5.7 Publish Reports

**JUnit report:**

```groovy
junit(
    testResults: 'test-results/junit.xml',
    allowEmptyResults: true
)
```

**HTML coverage report:**

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

Jenkins displays the reports when the configured output files exist.

## 5.8 Build React Application

```bash
set -eu
export CI=false
export NODE_OPTIONS=--openssl-legacy-provider
npm run build
```

This runs the application build script and expects output in the `build` directory.

## 5.9 Archive Artifacts

```groovy
archiveArtifacts(
    artifacts: 'build/**, coverage/**',
    allowEmptyArchive: true
)
```

The build and coverage files become available from the Jenkins build page.

# 6. Notifications

The `notifyStatus.groovy` function handles notifications.

## 6.1 Slack

The `slackSend()` step sends the build status, job name, build number, and console URL. Success and failure messages use different colors.

## 6.2 Email

The `emailext()` step sends build details and the Jenkins console URL. Notifications are triggered from the Jenkinsfile's `post` section.

# 7. Error Handling and Cleanup

| Mechanism | Purpose |
|---|---|
| `set -eu` | Stops shell execution on errors or unset variables |
| `|| true` | Suppresses test command failure |
| `allowEmptyResults` | Allows missing JUnit results |
| `allowMissing` | Allows missing HTML reports |
| `allowEmptyArchive` | Allows an empty archive |
| `post { success }` | Handles successful builds |
| `post { failure }` | Handles failed builds |
| `post { always }` | Runs final actions |
| `cleanWs()` | Cleans workspace files |

# 8. Conclusion

The shared library standardizes React CI by reusing dependency installation, testing, report publishing, build, artifact archiving, and notification functions.

# 9. Contact Information

| Name | Email |
|---|---|
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

# 10. References

- [Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)
- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [NodeJS Plugin](https://plugins.jenkins.io/nodejs/)
- [npm Documentation](https://docs.npmjs.com/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [JUnit Plugin](https://plugins.jenkins.io/junit/)
- [HTML Publisher Plugin](https://plugins.jenkins.io/htmlpublisher/)
- [Slack Plugin](https://plugins.jenkins.io/slack/)
- [Email Extension Plugin](https://plugins.jenkins.io/email-ext/)
- [Workspace Cleanup Plugin](https://plugins.jenkins.io/ws-cleanup/)
