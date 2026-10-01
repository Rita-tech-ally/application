<p align="center">
<img width="332" height="122" alt="image" src="https://github.com/user-attachments/assets/406e552d-c7f4-4842-a71e-28169405f712" />
</p>

---

# Golang Unit Testing

---

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 01/10/2026 | 1.0 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |

---

# Table of Contents

1. [Introduction](#1-introduction)
2. [Documentation](#2-documentation)
3. [What is Unit Testing?](#3-what-is-unit-testing)
4. [Why Unit Testing Matters](#4-why-unit-testing-matters)
5. [Workflow](#5-workflow)
6. [Different Golang Unit Testing Tools](#6-different-golang-unit-testing-tools)
7. [Tools Comparison](#7-tools-comparison)
8. [Advantages of Unit Testing](#8-advantages-of-unit-testing)
9. [Best Practices](#9-best-practices)
10. [Conclusion](#10-conclusion)
11. [Contact Information](#11-contact-information)
12. [References](#12-references)

---

# 1. Introduction

This document explains the concept of unit testing in Golang, its importance, testing workflow, commonly used testing tools, tool comparison, advantages, and best practices. It also explains how unit tests can be executed locally and integrated into a CI pipeline, with Go coverage tools used to measure code coverage.

---

# 2. Documentation

For the POC, refer to the official repository documentation:

[poc README](https://github.com/SnaatakAllStars/Sprint-2/blob/SCRUM-152-RITU/Documentation/Application_CI_Design/CI_Checks/Go/Unit_Testing/DOC/POC/README.md)

---

# 3. What is Unit Testing?

A unit test verifies whether a small and independent part of a Go application works as expected for a given input.

Go provides a built-in `testing` package for writing and executing unit tests. External dependencies can be replaced with mocks or stubs so that the actual unit can be tested in isolation.

| Characteristic | Description |
| --- | --- |
| **Isolated** | Tests a specific function or method independently. |
| **Fast** | Unit tests usually execute quickly. |
| **Deterministic** | Produces the same result for the same input and conditions. |
| **Automated** | Tests can be executed automatically using `go test`. |
| **Repeatable** | The same tests can be executed multiple times without manual effort. |

---

# 4. Why Unit Testing Matters

Unit testing helps developers identify problems early and verify that individual functions or components behave correctly.

| Without Unit Testing | With Unit Testing |
| --- | --- |
| **Bugs may be discovered during later testing or production.** | Bugs can be detected during development. |
| **Refactoring existing code can be risky.** | Tests provide a safety net during code changes. |
| **Expected behavior may not be clearly verified.** | Tests demonstrate expected behavior. |
| **Manual verification may be required repeatedly.** | Tests can run automatically. |
| **Feedback can take longer.** | Developers receive fast feedback. |

Unit testing therefore helps reduce debugging effort and makes code changes safer.

---

# 5. Workflow

<img width="5970" height="2220" alt="image" src="https://github.com/user-attachments/assets/dde292e6-0177-45da-bab1-477dbac5c98e" />

| Step | Description |
| --- | --- |
| **1. Write/Change Code** | Developer creates or modifies application logic. |
| **2. Write Unit Test** | Tests are created for the required behavior. |
| **3. Run Tests** | Tests are executed locally using `go test`. |
| **4. Check Result** | If a test fails, the code or test is reviewed and corrected. |
| **5. Commit & Push** | After successful local testing, code is pushed to the repository. |
| **6. CI Testing** | CI automatically executes the unit tests. |
| **7. Coverage** | Go coverage tools can generate a code coverage report. |
| **8. Continue Pipeline** | If required checks pass, the pipeline can continue with further stages. |

---

# 6. Different Golang Unit Testing Tools

| Tool | Category | Purpose | Common Usage |
| --- | --- | --- | --- |
| **Go testing Package** | Testing Framework | Built-in framework for writing and running Go unit tests. | `testing`, `TestXxx`, `t.Run()` |
| **Testify** | Testing Toolkit | Provides assertions, mocks, and test suites. | `assert.Equal()`, `require.NoError()` |
| **GoMock** | Mocking Framework | Generates and manages mocks for Go interfaces. | `gomock`, `EXPECT()` |
| **Go Coverage** | Code Coverage | Measures how much Go code is executed by tests. | `go test -cover` |
| **Ginkgo** | Testing Framework | Provides a behavior-driven testing framework for Go. | `Describe()`, `It()` |
| **GoConvey** | Testing Framework | Provides readable test specifications and web-based test output. | `Convey()`, `So()` |

---

# 7. Tools Comparison

| Tool | Main Purpose | Best Used For | Required? |
| --- | --- | --- | --- |
| **Go testing Package** | Writing and running tests | General Go unit testing | Recommended |
| **Testify** | Assertions and mocking | Readable assertions and test utilities | Optional |
| **GoMock** | Mocking dependencies | Testing code with interface-based dependencies | Optional |
| **Go Coverage** | Code coverage | Measuring tested code | Optional but useful |
| **Ginkgo** | Behavior-driven testing | BDD-style Go testing | Alternative |
| **GoConvey** | Test framework | Readable test specifications | Alternative |

---

# 8. Advantages of Unit Testing

| Advantage | Description |
| --- | --- |
| **Early Bug Detection** | Finds problems before they reach later testing stages. |
| **Safe Refactoring** | Existing behavior can be verified after code changes. |
| **Fast Feedback** | Go unit tests execute quickly and provide immediate results. |
| **Easy Debugging** | A failing test can help identify a specific function or behavior. |
| **Living Documentation** | Tests demonstrate the expected behavior of the code. |
| **CI/CD Integration** | Unit tests can automatically run as part of the CI pipeline. |
| **Improved Code Design** | Testable code encourages better separation of responsibilities. |

---

# 9. Best Practices

| # | Best Practice | Description |
| --- | --- | --- |
| **1** | **Test One Behavior** | Keep each unit test focused on a specific behavior or scenario. |
| **2** | **Use Table-Driven Tests** | Use test tables to efficiently test multiple inputs and expected outputs. |
| **3** | **Mock External Dependencies** | Mock databases, APIs, caches, and other external dependencies when required. |
| **4** | **Test Edge Cases** | Test empty values, boundary values, invalid inputs, and error conditions. |
| **5** | **Run Tests in CI** | Run unit tests automatically in the CI pipeline for relevant code changes. |

---

# 10. Conclusion

Unit testing helps verify individual functions, detect bugs early, and make code changes safer.

In Golang, the `testing` package is used for unit tests, while Testify and GoMock provide additional testing and mocking capabilities. Code coverage can be measured using Go's built-in coverage tools.

---

# 11. Contact Information

| Name | Email Address |
| --- | --- |
| Ritu | ritu.dogra.snaatak@mygurukulam.co |

---

# 12. References

| **Source** | **Reference Link** |
| --- | --- |
| **Go Testing Package** | [Go Testing Documentation](https://pkg.go.dev/testing) |
| **Go Test Command** | [Go Test Documentation](https://pkg.go.dev/cmd/go#hdr-Test_packages) |
| **Go Coverage** | [Go Coverage Documentation](https://go.dev/blog/integration-test-coverage) |
| **Testify** | [Testify Documentation](https://github.com/stretchr/testify) |
| **GoMock** | [GoMock Documentation](https://github.com/uber-go/mock) |
| **Ginkgo** | [Ginkgo Documentation](https://onsi.github.io/ginkgo/) |
| **GoConvey** | [GoConvey Documentation](https://smartystreets.github.io/goconvey/) |
