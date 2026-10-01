<p align="center">
<img width="332" height="122" alt="image" src="https://github.com/user-attachments/assets/406e552d-c7f4-4842-a71e-28169405f712" />
</p>

---

# POC of Golang Unit Testing

---

# Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 01/10/2026 | 1.0 | Liyakhat/Anirudh | Aman Raj | Sandeep Rawat/Ravindra |

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-pre-requisites)
3. [Documentation](#3-documentation)
4. [Golang Unit Testing Steps](#4-golang-unit-testing-steps)
5. [Flow](#5-flow)
6. [Conclusion](#6-conclusion)
7. [Contact Information](#7-contact-information)
8. [References](#8-references)

---

# 1. Introduction

This POC demonstrates how to perform Golang unit testing locally using the **Go standard testing framework, Testify, and Go native coverage tooling**. It covers application compilation, unit test execution, test failure detection, and code coverage generation.

The POC helps verify that the application logic works as expected and that its code is adequately tested in isolation before further development, deployment, or CI pipeline execution.

---

# 2. Pre-requisites

| Requirement | Configuration |
| --- | --- |
| **Cloud Platform** | AWS |
| **Service** | Amazon EC2 |
| **Operating System** | Ubuntu 24.04 LTS |
| **Go Compiler** | `go version` |
| **Git** | `git --version` |

---

# 3. Documentation

Documentation, refer to the official repository documentation:

[doc README](https://github.com/SnaatakAllStars/Sprint-2/blob/SCRUM-152-RITU/Documentation/Application_CI_Design/CI_Checks/Go/Unit_Testing/DOC/POC/README.md)

---

# 4. Golang Unit Testing Steps

## Step 4.1 — Set Up the Environment

Install Go if required:

```bash
sudo apt update
sudo apt install golang-go -y
```

Verify the installation:

```bash
go version
```

---

## Step 4.2 — Clone and Verify Project Structure

Clone the Golang Unit Testing POC repository:

```bash
git clone <repository-url>
cd employee-ci-poc
```

**Verify the project structure:**

```bash
tree
```

Expected structure:

```text
employee-ci-poc/
├── go.mod
├── go.sum
└── service/
    ├── employee.go
    └── employee_test.go
```

The project contains the application logic, interface definitions, unit test code, mocked test cases using Testify, and Go module dependency files.

---

## Step 4.3 — Compile the Application

Run:

```bash
go build ./...
```

This compiles all packages within the repository without running tests.

Expected result:

```text
Exits cleanly with status code 0 and no compilation errors.
```

---

## Step 4.4 — Run Unit Tests

Run:

```bash
go test -v ./...
```

Expected result:

```text
=== RUN   TestCreateEmployee_Success
--- PASS: TestCreateEmployee_Success (0.00s)
=== RUN   TestCreateEmployee_EmptyID
--- PASS: TestCreateEmployee_EmptyID (0.00s)
=== RUN   TestCreateEmployee_EmptyName
--- PASS: TestCreateEmployee_EmptyName (0.00s)
=== RUN   TestGetEmployee_Success
--- PASS: TestGetEmployee_Success (0.00s)
=== RUN   TestGetEmployee_NotFound
--- PASS: TestGetEmployee_NotFound (0.00s)
=== RUN   TestGetEmployee_EmptyID
--- PASS: TestGetEmployee_EmptyID (0.00s)
PASS
ok      employee-ci-poc/service 0.004s
```

All **6 test cases** pass without requiring an active database because the repository layer is mocked.

---

## Step 4.5 — Generate Go Coverage

Run:

```bash
go test -coverprofile=cover.out ./...
go tool cover -func=cover.out
```

Expected result:

```text
employee-ci-poc/service/employee.go:27:  NewEmployeeService  100.0%
employee-ci-poc/service/employee.go:32:  CreateEmployee      100.0%
employee-ci-poc/service/employee.go:43:  GetEmployee         100.0%
total:                                  (statements)        100.0%
```

The unit test suite provides **100% statement coverage** for the application code.

---

## Step 4.6 — View Go Coverage Report

Generate the HTML coverage report:

```bash
go tool cover -html=cover.out -o coverage.html
```

On a machine with a graphical browser:

```bash
xdg-open coverage.html
```

The report shows:

| Element | Description |
| --- | --- |
| **Green** | Executed and tested statement lines |
| **Red** | Unexecuted or uncovered code |
| **Grey** | Declarations, structs, comments, and non-executable syntax |

> **Note:** On a headless EC2 instance, `xdg-open` may not launch because no graphical browser is available. The HTML report is still generated at `coverage.html`.

The report can also be accessed using:

```bash
python3 -m http.server 8000
```

---

## Step 4.7 — Test Failure Scenario

To verify that the tests detect incorrect application behavior, temporarily modify the validation logic in:

```text
service/employee.go
```

For example:

```go
// Intentionally bypass validation logic
if strings.TrimSpace(emp.ID) == "" {
    return nil // Should return an error, returning nil instead
}
```

Run the tests:

```bash
go test ./...
```

Expected failure:

```text
--- FAIL: TestCreateEmployee_EmptyID (0.00s)
    employee_test.go:49:
        Error Trace: employee_test.go:49
        Error:       An error is expected but got nil.
FAIL
FAIL    employee-ci-poc/service    0.004s
FAIL
```

Restore the original valid code and run:

```bash
go test ./...
```

The test suite will return:

```text
PASS
```

This demonstrates that unit tests detect unexpected changes in application behavior.

---

# 5. Flow

```text
Golang Source Code
       ↓
go build ./...
       ↓
Compilation Successful
       ↓
go test -v ./...
       ↓
Go Testing + Testify Mocks
       ↓
Tests Passed
       ↓
go test -coverprofile=cover.out ./...
       ↓
Go Coverage Generation
       ↓
Coverage Report
       ↓
coverage.html
```

---

# 6. Conclusion

This POC demonstrates local Golang unit testing using the **Go standard testing package, Testify, and Go coverage tools**.

It verifies code compilation, unit test execution, mock-based isolation, failure detection, and code coverage without requiring external services or Jenkins CI.

---

# 7. Contact Information

| Name | Email Address |
| --- | --- |
| Ritu | [ritu.dogra.snaatak@mygurukulam.com](mailto:ritu.dogra.snaatak@mygurukulam.com) |

---

# 8. References

| **Source** | **Reference Link** |
| --- | --- |
| **Go Testing Package** | [Go Testing Documentation](https://pkg.go.dev/testing) |
| **Go Coverage Tool** | [Go Cover Documentation](https://pkg.go.dev/cmd/cover) |
| **Testify Toolkit** | [Testify GitHub Repository](https://github.com/stretchr/testify) |
