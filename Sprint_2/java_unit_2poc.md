<p align="center">
<img width="170" height="148" alt="image" src="https://github.com/user-attachments/assets/4a2ee792-2c62-44f6-b3bf-89bc5e1975b1" />
</p>

---

# POC of Java Unit Testing
---

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 30/09/2026 | 1.0 | Liyakhat/Anirudh  | Aman Raj | Sandeep Rawat/Ravindra |

---

## Pre-requisites

| **Tool**        | **Check Command**   | 
| ------------ | --------------- | 
| **Java JDK** | `java -version` | 
| **Maven**    | `mvn -version`  |


---

## Step 1 — Install

Install if required:

```bash
sudo apt install openjdk-17-jdk maven -y
```

Verify:

```bash
java -version
mvn -version
```

---

## Step 2 — Compile the Application

Run:

```bash
mvn clean compile
```

This compiles the application code without running tests.

Expected result:

```text
BUILD SUCCESS
```

---

## Step 3 — Run Unit Tests

Run:

```bash
mvn test
```

The POC uses:

* **JUnit 5** — for writing and running unit tests.
* **Mockito** — for mocking `SalaryRepository`.
* **Maven Surefire** — for executing tests.

Expected result:

```text
Tests run: 6, Failures: 0, Errors: 0, Skipped: 0

BUILD SUCCESS
```

Test reports are generated under:

```text
target/surefire-reports/
```

---

## Step 4 — Generate JaCoCo Coverage

Run:

```bash
mvn clean verify
```

This command:

1. Compiles the application.
2. Runs the unit tests.
3. Generates the JaCoCo coverage report.
4. Checks the configured **70% coverage threshold**.

Expected result:


---

## Step 5 — View JaCoCo Coverage Report

The report is generated at:

```text
target/site/jacoco/index.html
```

On a machine with a graphical browser:

```bash
xdg-open target/site/jacoco/index.html
```

The report shows:

| Coverage        | Description                    |
| --------------- | ------------------------------ |
| **Instruction** | Executed bytecode instructions |
| **Line**        | Executed source-code lines     |
| **Branch**      | Executed decision branches     |
| **Method**      | Executed methods               |
| **Class**       | Executed classes               |

> **Note:** On a headless EC2 server, `xdg-open` may not work because no browser is installed. The HTML report is still successfully generated at `target/site/jacoco/index.html`.

---

## Step 6 — Test Failure Scenario

To verify that the tests detect incorrect behavior, temporarily modify the salary calculation in:

```text
src/main/java/com/example/poc/SalaryService.java
```

Run:

```bash
mvn test
```

The test should fail:

```text
Tests run: 6, Failures: 1, Errors: 0

BUILD FAILURE
```

Restore the correct code and run:

```bash
mvn test
```

Expected result:

```text
BUILD SUCCESS
```

This demonstrates that unit tests detect unexpected changes in application behavior.

---

##  Flow

```text
Java Source Code
       ↓
mvn clean compile
       ↓
Compile
       ↓
mvn test
       ↓
JUnit 5 + Mockito
       ↓
Tests Passed
       ↓
mvn clean verify
       ↓
JaCoCo Coverage
       ↓
Coverage Report
```

---

## Conclusion

This POC demonstrates local Java unit testing using JUnit 5, Mockito, Maven, and JaCoCo.
It verifies code compilation, unit test execution, failure detection, and code coverage without using any CI/CD tool.

---
# 8. Contact Information

| Name | Email Address                                                                 |
| ---- | ----------------------------------------------------------------------------- |
| Ritu | [ritu.dogra.snaatak@mygurukulam.co](mailto:ritu.dogra.snaatak@mygurukulam.co) |

---

## References

| **Source**                           | **Reference Link**                                                                    |
| ------------------------------------ | ------------------------------------------------------------------------------------- |
| **JUnit 5 – User Guide**             | [JUnit 5 Official Documentation](https://docs.junit.org/5.10.0/user-guide/index.html) |
| **Mockito – Official Documentation** | [Mockito Official Documentation](https://site.mockito.org/)                           |
| **Maven – Official Documentation**   | [Apache Maven Documentation](https://maven.apache.org/guides/)                        |
| **JaCoCo – Official Documentation**  | [JaCoCo Official Documentation](https://www.jacoco.org/jacoco/trunk/doc/)             |


