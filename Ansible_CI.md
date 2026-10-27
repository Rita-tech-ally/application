
# Jenkins Shared Library for React CI Pipeline

## Document Information

| Author | Created On | Version |
|---|---|---|
| Ritu | 25/10/2026 | 1.0 |

## 1. Introduction

This Jenkins Shared Library automates the Continuous Integration (CI) process for React applications using Node.js and npm. It provides reusable functions for dependency installation, testing, coverage reporting, application builds, artifact archiving, and notifications.

## 2. Tools and Technologies

- **Jenkins:** Pipeline automation
- **Groovy:** Shared Library implementation
- **Node.js 18:** JavaScript runtime
- **npm:** Dependency management and build execution
- **Jest:** Test execution and coverage generation
- **JUnit:** Test report publishing
- **Slack and Email:** Build notifications
- **GitHub:** Source code management

## 3. Library Workflow

```text
Checkout Frontend Repository
          |
          v
       Call reactCI()
          |
          v
 Configure Node.js and npm
          |
          v
   Install Dependencies
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
```

## 4. Implementation

The Jenkinsfile loads the shared library from GitHub using the `SCRUM-258-RITU` branch and `Github-cred` credentials.

The reusable `reactCI()` function accepts the application directory, Node.js tool name (`NodeJS_18`), and test execution option.

The library installs dependencies using `npm ci` when `package-lock.json` exists; otherwise, it uses `npm install`. It runs tests, publishes JUnit XML and HTML coverage reports, builds the application, and archives generated artifacts.

## 5. Notifications and Cleanup

The `notifyStatus()` function sends build status notifications through Slack and email. Jenkins uses `post` conditions to handle successful and failed builds. The `cleanWs()` step removes workspace files after execution.

**Note:** Test failures may not fail the pipeline because the test command uses `|| true`.

## 6. References

- [Jenkins Shared Libraries](https://www.jenkins.io/doc/book/pipeline/shared-libraries/)
- [npm Documentation](https://docs.npmjs.com/)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
