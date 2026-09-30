# POC Run Guide — salary-ci-poc

This guide explains how to run the Java Unit Testing POC locally using **JUnit 5, Mockito, Maven, and JaCoCo**. The POC demonstrates compilation, unit test execution, and code coverage generation on a local machine.

---

## Pre-requisites

| Tool         | Check Command   | Version Needed |
| ------------ | --------------- | -------------- |
| **Java JDK** | `java -version` | 17+            |
| **Maven**    | `mvn -version`  | 3.6+           |

Install the required tools if they are not already available:

```bash
sudo apt install openjdk-17-jdk maven -y
```

Verify the installation:

```bash
java -version
mvn -version
```

---

## Step 1 — Extract the POC

Extract the POC ZIP file and move into the project directory:

```bash
unzip salary-ci-poc.zip
cd salary-ci-poc
```

Check the project structure:

```bash
ls -R
```

Expected structure:

```text
salary-ci-poc/
├── pom.xml
├── README.md
├── src/
│   ├── main/
│   │   └── java/com/example/poc/
│   │       ├── Employee.java
│   │       ├── SalaryRepository.java
│   │       └── SalaryService.java
│   └── test/
│       └── java/com/example/poc/
│           └── SalaryServiceTest.java
```

---

## Step 2 — Compile the Application

Run:

```bash
mvn clean compile
```

This command compiles the application source code from `src/main`.

It does **not** execute the unit tests.

Expected result:

```text
BUILD SUCCESS
```

A successful compilation confirms that the application code has no compilation errors.

---

## Step 3 — Run Unit Tests

Run:

```bash
mvn test
```

This executes the unit tests written in `SalaryServiceTest`.

The POC uses:

* **JUnit 5** — to create and execute unit tests.
* **Mockito** — to mock `SalaryRepository`.
* **Maven Surefire Plugin** — to execute the tests.

No real database is required because the repository dependency is mocked.

Expected result:

```text
Tests run: 6, Failures: 0, Errors: 0, Skipped: 0

BUILD SUCCESS
```

---

## Step 4 — Run Tests with JaCoCo Coverage

Run:

```bash
mvn clean verify
```

This command performs the complete local verification:

1. Cleans previous build files.
2. Compiles the application.
3. Executes the unit tests.
4. Generates the JaCoCo coverage report.
5. Checks whether the configured coverage threshold is satisfied.

The POC is configured with a **70% coverage threshold**.

Expected result:

```text
Tests run: 6, Failures: 0, Errors: 0

BUILD SUCCESS
```

If the coverage is below the configured threshold, the Maven build fails.

---

## Step 5 — View the JaCoCo Coverage Report

After running:

```bash
mvn clean verify
```

the JaCoCo report is generated at:

```text
target/site/jacoco/index.html
```

Open the report in a browser.

### Linux

```bash
xdg-open target/site/jacoco/index.html
```

The report displays coverage information such as:

| Coverage Type   | Description                                  |
| --------------- | -------------------------------------------- |
| **Instruction** | Percentage of executed bytecode instructions |
| **Line**        | Percentage of executed source-code lines     |
| **Branch**      | Percentage of executed decision branches     |
| **Method**      | Percentage of executed methods               |
| **Class**       | Percentage of classes executed by tests      |

The `SalaryService` class can be checked to see how much of its business logic is covered by the unit tests.

---

## Step 6 — Check Test Reports

Maven also generates test execution reports under:

```text
target/surefire-reports/
```

List the generated reports:

```bash
ls target/surefire-reports/
```

These reports contain the results of the executed unit tests.

---

## Step 7 — Optional Checkstyle

If Checkstyle is configured in the project, it can be executed locally using:

```bash
mvn checkstyle:checkstyle
```

The generated report can be checked under:

```text
target/
```

This step is optional and is separate from unit test execution.

---

## Step 8 — Test Failure Demonstration

To verify that the test suite can detect incorrect behavior, temporarily modify the business logic in:

```text
src/main/java/com/example/poc/SalaryService.java
```

For example, change the expected salary calculation.

Run:

```bash
mvn test
```

The test should fail.

Example:

```text
Tests run: 6, Failures: 1, Errors: 0

BUILD FAILURE
```

Restore the correct code and run:

```bash
mvn test
```

The tests should pass again.

This demonstrates how unit tests detect changes that break the expected application behavior.

---

## Troubleshooting

| Problem                                | Cause                                               | Fix                                                     |
| -------------------------------------- | --------------------------------------------------- | ------------------------------------------------------- |
| `mvn: command not found`               | Maven is not installed                              | `sudo apt install maven -y`                             |
| `java: command not found`              | Java is not installed                               | `sudo apt install openjdk-17-jdk -y`                    |
| Tests fail with `NullPointerException` | Mock is not initialized                             | Check `@Mock` and `@ExtendWith(MockitoExtension.class)` |
| `BUILD FAILURE` during coverage        | Coverage is below 70%                               | Add tests for uncovered code                            |
| JaCoCo report not found                | `verify` was not executed                           | Run `mvn clean verify`                                  |
| Unit test failure                      | Application behavior does not match expected result | Check the implementation and test case                  |

---

## POC Flow

```text
Java Source Code
       ↓
mvn clean compile
       ↓
Compile Successful
       ↓
mvn test
       ↓
JUnit 5 + Mockito Tests
       ↓
Tests Passed
       ↓
mvn clean verify
       ↓
JaCoCo Coverage
       ↓
Coverage Threshold Check
       ↓
Coverage Report
```

---

## Summary

This POC demonstrates how Java unit testing can be performed completely on a local machine.

The main commands used are:

```bash
mvn clean compile
mvn test
mvn clean verify
```

The POC uses **JUnit 5** for unit testing, **Mockito** for mocking dependencies, **Maven** for build and test execution, and **JaCoCo** for code coverage.

No Jenkins or other CI/CD tool is required for this POC.
