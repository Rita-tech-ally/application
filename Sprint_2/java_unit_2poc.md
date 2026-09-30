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
## Introduction

This POC demonstrates how to perform Java unit testing locally using **JUnit 5, Mockito, Maven, and JaCoCo**. It covers application compilation, unit test execution, test failure detection, and code coverage generation.

The POC helps verify that the application works as expected and that its code is adequately tested before further development or deployment.

---
## Table of Contents

1. [Introduction](#introduction)
2. [Pre-requisites](#pre-requisites)
3. [Step 1 — Set Up the Environment](#step-1--set-up-the-environment)
4. [Step 2 — Clone and Verify Project Structure](#step-2--clone-and-verify-project-structure)
5. [Step 3 — Compile the Application](#step-3--compile-the-application)
6. [Step 4 — Run Unit Tests](#step-4--run-unit-tests)
7. [Step 5 — Generate JaCoCo Coverage](#step-5--generate-jacoco-coverage)
8. [Step 6 — View JaCoCo Coverage Report](#step-6--view-jacoco-coverage-report)
9. [Step 7 — Test Failure Scenario](#step-7--test-failure-scenario)
10. [Flow](#flow)
11. [Conclusion](#conclusion)
12. [Contact Information](#8-contact-information)
13. [References](#references)


---


## Pre-requisites


| Requirement      | Configuration        |
| ---------------- | -------------------- |
| **Cloud Platform**   | AWS                  |
| **Service**          | Amazon EC2           |
| **Operating System** | Ubuntu 24.04 LTS     |
| **Java JDK** | `java -version` | 
| **Maven**    | `mvn -version`  |


---


## Step 1 — Set Up the Environment

Install if required:

```bash
sudo apt install openjdk-17-jdk maven -y
```

Verify:

```bash
java -version
mvn -version
```
<img width="856" height="240" alt="Screenshot from 2026-09-30 22-40-05" src="https://github.com/user-attachments/assets/18a990f4-1a08-41b9-890b-205910d61ff0" />


---


## Step 2 — Clone and Verify Project Structure

Clone the Java Unit Testing POC repository:

```bash
git clone <repository-url>
```

<img width="847" height="141" alt="git clone _" src="https://github.com/user-attachments/assets/68bc8c98-670a-4635-be30-6ede6347b127" />

**Verify the project structure:**

```bash
tree
```
<img width="854" height="383" alt="tree_java" src="https://github.com/user-attachments/assets/e5ef4881-ffe2-43e1-9109-10db71775e6b" />


The project contains the application source code, unit test code, and `pom.xml` configuration file required to build and test the application.


---


## Step 3 — Compile the Application

Run:

```bash
mvn clean compile
```

This compiles the application code without running tests.

<img width="1626" height="201" alt="Screenshot from 2026-09-30 22-49-27" src="https://github.com/user-attachments/assets/74552c0f-c04a-4436-90a7-54c4a665fa97" />
<img width="1626" height="201" alt="Screenshot from 2026-09-30 22-50-03" src="https://github.com/user-attachments/assets/4d3cc4b6-9e16-4f5d-b0c5-2a72600a2f1a" />



---


## Step 4 — Run Unit Tests

Run:

```bash
mvn test
```
<img width="1661" height="193" alt="Screenshot from 2026-09-30 22-51-20" src="https://github.com/user-attachments/assets/5216a026-a975-44c2-aadb-3c1c21dff80a" />
<img width="1833" height="382" alt="Screenshot from 2026-09-30 22-52-07" src="https://github.com/user-attachments/assets/77c6588a-37c3-4196-b185-569d86c3f43e" />
<img width="1828" height="432" alt="Screenshot from 2026-09-30 22-52-51" src="https://github.com/user-attachments/assets/1012d3aa-1774-42bb-a913-0be490fc535f" />


Test reports are generated under:

```text
target/surefire-reports/
```
<img width="1151" height="162" alt="Screenshot from 2026-09-30 22-58-00" src="https://github.com/user-attachments/assets/277feb65-47e6-46a3-8f9f-a56fb1a81a92" />

---

## Step 5 — Generate JaCoCo Coverage

Run:

```bash
mvn clean verify
```
<img width="1836" height="793" alt="Screenshot from 2026-09-30 23-00-52" src="https://github.com/user-attachments/assets/a09659e4-3e51-4d1e-bc52-de31b1167622" />
<img width="1839" height="260" alt="Screenshot from 2026-09-30 23-01-19" src="https://github.com/user-attachments/assets/ea66eda2-92d3-4dcf-a866-f0afd36a70cd" />


---


## Step 6 — View JaCoCo Coverage Report

The report is generated at:

```text
target/site/jacoco/index.html
```
<img width="1832" height="258" alt="Screenshot from 2026-09-30 23-37-20" src="https://github.com/user-attachments/assets/f7540b38-9577-482a-86a3-194a159a7e9b" />

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

## Step 7 — Test Failure Scenario

To verify that the tests detect incorrect behavior, temporarily modify the salary calculation in:

```text
src/main/java/com/example/poc/SalaryService.java
```

Run:

```bash
mvn test
```

<img width="1840" height="788" alt="Screenshot from 2026-09-30 23-14-23" src="https://github.com/user-attachments/assets/d6e80eee-0cdc-4d2f-a595-487f73faaa2b" />
<img width="1839" height="768" alt="Screenshot from 2026-09-30 23-14-55" src="https://github.com/user-attachments/assets/6a0e3004-b663-418e-b6ff-5379050ec010" />


Restore the correct code and run:

```bash
mvn test
```
<img width="1835" height="784" alt="image" src="https://github.com/user-attachments/assets/11fb7c73-40c1-4dff-a35d-49fb7b7025e2" />


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


