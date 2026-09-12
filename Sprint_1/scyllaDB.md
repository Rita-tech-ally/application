# ScyllaDB Documentation

<p align="center">
  <img width="128" height="128" alt="ScyllaDB Icon" src="https://github.com/user-attachments/assets/3aee4f01-f9f5-49ea-b288-a6d4004d456c" />
</p>

---

## Document Information

| Author | Created On | Version | L0 Reviewer | L1 Reviewer | L2 Reviewer |
| --- | --- | --- | --- | --- | --- |
| Ritu | 09/08/2026 | 1.1 | Liyakhat | Aman Raj | Sandeep Rawat/Ravindra |

---

# Table of Contents

1. [Purpose](#1-purpose)
2. [Key Features](#2-key-features)
3. [Getting Started](#3-getting-started)
4. [How to Setup/Install ScyllaDB](#3-how-to-setupinstall-scylladb)
5. [Configuration](#4-configuration)
6. [Basic CQL Operations](#4-basic-cql-operations)
7. [Maintenance](#5-maintenance)
8. [Monitoring](#6-monitoring)
9. [Disaster Recovery](#7-disaster-recovery)
10. [High Availability](#8-high-availability)
11. [Conclusion](#9-conclusion)
12. [FAQs](#10-faqs)
13. [Contact Information](#11-contact-information)
14. [References](#12-references)

---

# 1. Purpose

The purpose of this document is to provide a structured guide for ScyllaDB installation, configuration, CQL operations, maintenance, monitoring, disaster recovery, and high availability on AWS EC2.

It covers the required prerequisites, dependencies, system requirements, important ports, installation steps, configuration parameters, database testing, and cluster health verification.

---

# 2. Key Features

| **Feature**              | **Description**                                                  |
| ------------------------ | ---------------------------------------------------------------- |
| High Performance         | Provides high throughput and low-latency database operations.    |
| Distributed Architecture | Supports data distribution across multiple nodes.                |
| Horizontal Scaling       | Allows additional nodes to be added as workload increases.       |
| High Availability        | Supports multi-node clusters across availability zones.          |
| CQL Support              | Provides Cassandra Query Language (CQL) for database operations. |
| Fault Tolerance          | Data can be replicated across multiple nodes.                    |
| AWS Support              | Can be deployed on AWS EC2 instances.                            |

---

# 3. Getting Started

##  Prerequisites

Before installing ScyllaDB, ensure the following prerequisites are available:

| **Requirement**      | **Details / Verification**                               |
| -------------------- | -------------------------------------------------------- |
| AWS EC2              | EC2 instance with supported Linux distribution.          |
| Operating System     | Ubuntu 24.04 LTS or Ubuntu 22.04 LTS.                    |
| Network Connectivity | Nodes should be able to communicate through private IPs. |
| Sudo Access          | Required for package installation and configuration.     |
| Security Group       | Required ScyllaDB ports must be allowed.                 |
| Time Synchronization | Ensure system time is synchronized.                      |
| Private IP           | Required for cluster communication between nodes.        |

---

## Software Overview

| **Component / Command** | **Purpose**                                                       |
| ----------------------- | ----------------------------------------------------------------- |
| `scylla-server`         | Main ScyllaDB database service.                                   |
| `scylla.yaml`           | Main ScyllaDB configuration file.                                 |
| `cqlsh`                 | Command-line interface for executing CQL queries.                 |
| `nodetool`              | Used for cluster administration and health checks.                |
| `scylla_dev_mode_setup` | Configures ScyllaDB for development or low-resource environments. |
| `systemctl`             | Used to start, stop, enable, and check the ScyllaDB service.      |

---

## System Requirements

The setup is performed on AWS EC2 using Ubuntu.

| **Requirement**   | **Environment**                                      |
| ----------------- | ---------------------------------------------------- |
| Platform          | AWS EC2                                              |
| OS                | Ubuntu 24.04 LTS / Ubuntu 22.04 LTS                  |
| Architecture      | Single-node POC or multi-node cluster                |
| Cluster IP        | Private IP address                                   |
| ScyllaDB Version  | ScyllaDB 5.4 / stable Open Source release            |
| Development Setup | `--smp 1` can be used for low-resource POC instances |

> For production deployments, hardware sizing should be based on workload, data size, throughput, and ScyllaDB recommendations.

---

##  Important Ports

| **Port** | **Protocol** | **Purpose**                                    |
| -------- | ------------ | ---------------------------------------------- |
| `9042`   | TCP          | Native CQL client and application connections. |
| `7000`   | TCP          | Inter-node cluster communication and gossip.   |

> Port `7000` should be accessible only between ScyllaDB cluster nodes. Port `9042` should be restricted to trusted clients or application networks.

---

# 2. Dependencies

The following packages are required during the installation:

| **Dependency** | **Purpose**                                          |
| -------------- | ---------------------------------------------------- |
| `curl`         | Downloads the official ScyllaDB installation script. |
| `gnupg`        | Handles package signing keys.                        |
| `python3`      | Required by ScyllaDB tooling and setup components.   |
| `apt`          | Installs and manages ScyllaDB packages.              |
| `systemd`      | Manages the ScyllaDB service.                        |

Install the basic dependencies:

```bash
sudo apt-get update
sudo apt-get install -y curl gnupg python3
```

---

## Other Dependencies

For a multi-node ScyllaDB cluster, the following additional requirements are needed:

* AWS Security Groups must allow required cluster ports.
* Nodes must communicate using private IP addresses.
* Each node should have a unique private IP.
* All nodes should use the appropriate ScyllaDB configuration.
* The same seed node should be configured for joining nodes.
* Nodes should be distributed across Availability Zones for better fault tolerance.

---

# 3. How to Setup/Install ScyllaDB

## 3.1 Clean Existing Repository Configuration

Remove previous ScyllaDB repository configurations:

```bash
sudo rm -f /etc/apt/sources.list.d/scylla*
sudo apt-get update
```

---

## 3.2 Install Required Packages

```bash
sudo apt-get install -y curl gnupg python3
```

---

## 3.3 Add ScyllaDB Repository

Run the official ScyllaDB installer:

```bash
curl -sSf https://get.scylladb.com/server | sudo bash
```

---

## 3.4 Configure the Repository Signing Key

```bash
sudo gpg --homedir /tmp --no-default-keyring \
--keyring /tmp/temp.gpg \
--export C503C686B007F39E | \
sudo tee /etc/apt/keyrings/scylladb.gpg > /dev/null
```

Copy the key:

```bash
sudo cp /etc/apt/keyrings/scylladb.gpg \
/etc/apt/trusted.gpg.d/scylladb.gpg
```

Set the required permissions:

```bash
sudo chmod 644 \
/etc/apt/keyrings/scylladb.gpg \
/etc/apt/trusted.gpg.d/scylladb.gpg
```

Update the package list:

```bash
sudo apt-get update
```

---

## 3.5 Install ScyllaDB

```bash
sudo apt-get install -y scylla
```

---

## 3.6 Configure Development Mode

For a low-resource EC2 POC environment:

```bash
sudo scylla_dev_mode_setup --developer-mode 1
```

Configure one CPU shard:

```bash
echo 'CPUSET="--smp 1"' | sudo tee /etc/scylla.d/cpuset.conf
```

Reload systemd:

```bash
sudo systemctl daemon-reload
```

---

## 3.7 Start and Enable ScyllaDB

Enable the service:

```bash
sudo systemctl enable scylla-server
```

Start the service:

```bash
sudo systemctl start scylla-server
```

Check the service:

```bash
sudo systemctl status scylla-server --no-pager
```

---

## 3.8 Verify Cluster Status

Check the ScyllaDB cluster:

```bash
nodetool status
```

A healthy node should show the **UN** status:

* **U** = Up
* **N** = Normal

---

# 4. Configuration

The main ScyllaDB configuration file is:

```text
/etc/scylla/scylla.yaml
```

Edit the configuration:

```bash
sudo nano /etc/scylla/scylla.yaml
```

Example configuration:

```yaml
cluster_name: 'Production-POC-Cluster'

seed_provider:
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          - seeds: "172.31.3.157"

listen_address: 172.31.3.157

rpc_address: 0.0.0.0

endpoint_snitch: Ec2Snitch
```

## Configuration Parameters

| **Parameter**     | **Purpose**                                                 |
| ----------------- | ----------------------------------------------------------- |
| `cluster_name`    | Defines the name of the ScyllaDB cluster.                   |
| `seed_provider`   | Defines the seed node used for cluster discovery.           |
| `listen_address`  | Defines the private IP used for node-to-node communication. |
| `rpc_address`     | Defines the address used for client connections.            |
| `endpoint_snitch` | Helps ScyllaDB understand the AWS infrastructure topology.  |

Restart ScyllaDB after configuration changes:

```bash
sudo systemctl restart scylla-server
```

---

# 4. Basic CQL Operations

After installing ScyllaDB, basic CQL queries can be used to verify database connectivity, create a keyspace and table, insert data, and retrieve records.

## 4.1 Connect to ScyllaDB

Connect to the ScyllaDB CQL shell using:

```bash
cqlsh localhost 9042
```

## 4.2 Create Keyspace

Create a keyspace using `NetworkTopologyStrategy`:

```sql
CREATE KEYSPACE poc_keyspace 
WITH replication = {'class': 'NetworkTopologyStrategy', 'datacenter1': 1};
```

> **Note:** Replication factor `1` is suitable for the current single-node POC environment. For a multi-node production cluster, the replication factor should be configured according to the cluster design.

## 4.3 Create Table

Create an `audit_log` table:

```sql
CREATE TABLE poc_keyspace.audit_log (
    event_id uuid PRIMARY KEY,
    service_name text,
    status text,
    created_at timestamp
);
```

## 4.4 Insert Test Record

Insert a test record into the table:

```sql
INSERT INTO poc_keyspace.audit_log 
(event_id, service_name, status, created_at)
VALUES 
(uuid(), 'user-auth-service', 'SUCCESS', toTimestamp(now()));
```

## 4.5 Query the Record

Retrieve the inserted record:

```sql
SELECT * FROM poc_keyspace.audit_log;
```

## 4.6 Verified Output

The query was successfully verified on the EC2 instance:

```text
 event_id                             | created_at                      | service_name      | status
--------------------------------------+---------------------------------+-------------------+---------
 04fcdf6d-0eec-4e52-bffd-1d5e1ff95618 | 2026-09-10 19:57:05.960000+0000 | user-auth-service | SUCCESS

(1 rows)
```

This confirms that **ScyllaDB is running, CQL connectivity is working, the keyspace and table were created successfully, and data can be inserted and retrieved successfully.**


---

# 5. Maintenance

Regular maintenance helps keep the ScyllaDB cluster healthy and reliable.

| **Task**               | **Command / Action**                   |
| ---------------------- | -------------------------------------- |
| Check Service          | `sudo systemctl status scylla-server`  |
| Restart Service        | `sudo systemctl restart scylla-server` |
| Check Cluster          | `nodetool status`                      |
| Check CQL              | `cqlsh localhost 9042`                 |
| Check Disk             | `df -h`                                |
| Check System Resources | `free -h` / `nproc`                    |
| Check Logs             | `sudo journalctl -u scylla-server`     |

---

# 6. Monitoring

Monitoring helps identify performance problems, resource exhaustion, node failures, and service availability issues.

| **Metric / Check** | **Purpose**                      | **Command / Tool**                    |
| ------------------ | -------------------------------- | ------------------------------------- |
| Node Status        | Detect failed or unhealthy nodes | `nodetool status`                     |
| CPU Usage          | Detect CPU saturation            | `top` / `htop`                        |
| Memory Usage       | Detect memory pressure           | `free -h` / `top`                     |
| Disk Usage         | Prevent storage exhaustion       | `df -h`                               |
| Disk I/O           | Identify storage bottlenecks     | `iostat`                              |
| Network Traffic    | Detect network saturation        | `ss -s` / `sar -n DEV`                |
| Service Status     | Verify service health            | `sudo systemctl status scylla-server` |
| CQL Port           | Verify port 9042                 | `ss -lntp \| grep 9042`               |
| Service Logs       | View recent logs                 | `journalctl -u scylla-server -n 100`  |
| Live Logs          | Monitor logs continuously        | `journalctl -u scylla-server -f`      |


---

# 7. Disaster Recovery

Disaster Recovery (DR) consists of processes, strategies, and tools used to recover ScyllaDB services and data after unexpected failures or disasters.

| **Failure Scenario**      | **Protection Mechanism**          |
| ------------------------- | --------------------------------- |
| Node Failure              | Data replication                  |
| Disk Failure              | Replicated data and backups       |
| Data Corruption           | Backup and restore                |
| Accidental Deletion       | Backup                            |
| Availability Zone Failure | Multi-AZ deployment               |
| Datacenter Failure        | Multi-DC deployment               |
| Region Failure            | Cross-region strategy and backups |

---

# 8. High Availability

High Availability (HA) ensures that ScyllaDB remains accessible with minimal downtime even when individual infrastructure components fail.

| **HA Component**   | **Recommendation**                             |
| ------------------ | ---------------------------------------------- |
| Cluster            | Use multiple nodes                             |
| Replication        | Configure an appropriate replication factor    |
| Availability Zones | Distribute nodes across AZs                    |
| Datacenters        | Use multiple DCs where required                |
| Consistency        | Select according to application requirements   |
| Backups            | Maintain independent backups                   |
| Monitoring         | Configure health monitoring and alerts         |
| Capacity           | Maintain sufficient capacity for node failures |


# 9. Conclusion

ScyllaDB provides high performance, scalability, and high availability for modern applications. A properly configured ScyllaDB deployment improves database performance, reliability, and fault tolerance.

# 10. FAQs

### Is ScyllaDB Cassandra compatible?

Yes. ScyllaDB supports CQL and Cassandra-compatible clients and applications.

###  Does ScyllaDB support DynamoDB?

Yes. ScyllaDB provides the Alternator API, which provides DynamoDB-compatible access.


###  Does ScyllaDB support multiple datacenters?

Yes. ScyllaDB supports multi-datacenter deployments and topology-aware replication.

---

# 11. Contact Information

| Name |         Email Address             |
| ---- | ----------------------------------|
| Ritu | ritu.dogra.snaatak@mygurukulam.co |

---

# 12. References

| **Reference**                                                                                                           | **Purpose**                         |
| ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| [ScyllaDB Documentation](https://docs.scylladb.com/manual/stable/)                                                      | Official ScyllaDB documentation     |
| [ScyllaDB Installation](https://docs.scylladb.com/manual/stable/getting-started/install-scylla/)                        | ScyllaDB installation procedures    |
| [ScyllaDB System Requirements](https://docs.scylladb.com/manual/stable/getting-started/system-requirements.html)        | Hardware and platform requirements  |
| [ScyllaDB Features](https://docs.scylladb.com/manual/stable/features/)                                                  | ScyllaDB feature 
---
