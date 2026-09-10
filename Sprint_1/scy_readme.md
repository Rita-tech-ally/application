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

## Table of Contents

* [1. Purpose](#1-purpose)
* [2. Key Features](#2-key-features)
* [3. Prerequisites](#3-prerequisites)
* [4. Dependencies](#4-dependencies)
* [5. How to Setup/Install ScyllaDB](#5-how-to-setupinstall-scylladb)
* [6. Configuration](#6-configuration)
* [7. Basic CQL Operations](#7-basic-cql-operations)
* [8. Maintenance](#8-maintenance)
* [9. Monitoring](#9-monitoring)
* [10. Disaster Recovery](#10-disaster-recovery)
* [11. High Availability](#11-high-availability)
* [12. Troubleshooting](#12-troubleshooting)
* [13. FAQs](#13-faqs)
* [14. Contact Information](#14-contact-information)
* [15. References](#15-references)


---

# 1. Purpose
The purpose of this documentation is to provide a single reference for ScyllaDB installation, configuration, operations, monitoring, troubleshooting.

---

# 2. Key Features

| **Feature**              | **Description**                                                                                |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| Distributed Architecture | Data is distributed across multiple nodes in the cluster.                                      |
| High Performance         | Uses the Seastar framework and shard-per-core architecture for efficient resource utilization. |
| Horizontal Scaling       | Capacity can be increased by adding nodes to the cluster.                                      |
| Cassandra Compatibility  | Supports CQL and Cassandra-compatible clients and applications.                                |
| DynamoDB Compatibility   | Provides the Alternator API for DynamoDB-compatible applications.                              |
| Replication              | Supports replication of data across nodes and datacenters.                                     |
| Fault Tolerance          | Allows the cluster to continue operating during node failures.                                 |
| Configurable Consistency | Supports different consistency levels according to application requirements.                   |
| Change Data Capture      | Captures database changes for downstream processing.                                           |
| Vector Search            | Supports similarity search using vector embeddings.                                            |
| Full-Text Search         | Supports text-based search workloads.                                                          |
| Lightweight Transactions | Supports conditional operations requiring stronger consistency.                                |
| Backup and Restore       | Provides backup and recovery capabilities for protecting data.                                 |

---


# 3. Prerequisites

| Requirement          | Verification                        |
| -------------------- | ----------------------------------- |
| Supported Linux OS   | `cat /etc/os-release`               |
| CPU & Memory         | `nproc` / `free -h`                 |
| Disk Space           | `df -h`                             |
| Network Connectivity | `ping <node-ip>`                    |
| SSH Access           | `ssh -V`                            |
| Sudo Access          | `sudo -v`                           |
| Required Tools       | `curl --version` / `wget --version` |

### 3.1 Software Overview

ScyllaDB is a distributed NoSQL wide-column database compatible with Cassandra applications. It uses **CQL** as its primary query language and also supports the **Alternator API**.

| **Component / File**          | **Purpose**                       |
| ----------------------------- | --------------------------------- |
| `scylla-server`               | Main ScyllaDB service             |
| `scylla.yaml`                 | Main configuration                |
| `cassandra-rackdc.properties` | Datacenter and rack configuration |
| `cqlsh`                       | CQL command-line interface        |
| `nodetool`                    | Cluster administration            |
| `scylla_setup`                | Initial setup and configuration   |


## 3.2 System Requirements

| **Resource**             | **Recommendation / Requirement**                            |
| ------------------------ | ----------------------------------------------------------- |
| CPU                      | Modern CPU with SSE4.2 support                              |
| CPU Cores                | Workload dependent; larger workloads require more CPU cores |
| RAM                      | 16 GB or 2 GB per logical core, whichever is higher         |
| Medium/High Workload RAM | Approximately 64–256 GB depending on workload               |
| Storage                  | SSD strongly recommended                                    |
| Disk/RAM Ratio           | Approximately 30:1 as a general sizing guideline            |
| Filesystem               | XFS recommended                                             |
| Network                  | 10 Gbps or higher recommended for large nodes               |

> Exact production sizing should be determined based on workload, dataset size, replication factor, throughput, latency requirements, and expected growth.

---

## 3.3 Important Ports

| **Port** | **Protocol** | **Purpose**                |
| -------: | ------------ | -------------------------- |
|       22 | TCP          | SSH administration         |
|     7000 | TCP          | Node-to-node communication |
|     9042 | TCP          | CQL client connections     |
|     9142 | TCP          | TLS CQL connections        |
|    10000 | TCP          | REST/management access     |

> Restrict ScyllaDB ports using firewall rules and security groups. Do not expose internal ports to the public internet.
---

# 4. Dependencies

## 4.1 Runtime Dependencies

| **Dependency** | **Purpose**                               |
| -------------- | ----------------------------------------- |
| Linux          | Operating system required to run ScyllaDB |
| XFS            | Recommended production filesystem         |
| systemd        | Service and process management            |
| TCP/IP         | Client and cluster communication          |
| CQL Driver     | Application connectivity with ScyllaDB    |

---

## 4.2 Other Dependencies

| **Dependency / Tool** | **Purpose**                                         |
| --------------------- | --------------------------------------------------- |
| `cqlsh`               | Execute CQL queries and manage database schemas     |
| `nodetool`            | Perform node and cluster administration             |
| `scylla_setup`        | Configure system, storage, and ScyllaDB settings    |
| ScyllaDB Manager      | Manage ScyllaDB clusters and operational tasks      |
| Monitoring Stack      | Collect metrics and configure monitoring and alerts |
| Object Storage        | Store database backups                              |

---

# 5. How to Setup/Install ScyllaDB

## 5.1 Step-by-step Installation Instructions

### Step 1: Install ScyllaDB

Add the ScyllaDB repository:

```bash
curl -sSf https://repositories.scylladb.com/scylla/repo/$(lsb_release -sc)/scylladb-2026.1.list | sudo tee /etc/apt/sources.list.d/scylla.list
```

Update the package index:

```bash
sudo apt update
```

Install ScyllaDB:

```bash
sudo apt install scylla -y
```

### Step 2: Verify Installation

```bash
scylla --version
```

### Step 3: Run Initial Setup

```bash
sudo scylla_setup
```

### Step 4: Start ScyllaDB

```bash
sudo systemctl start scylla-server
```

### Step 5: Enable ScyllaDB at Boot

```bash
sudo systemctl enable scylla-server
```

### Step 6: Verify Service Status

```bash
sudo systemctl status scylla-server
```

The service should show `active (running)`.

### Step 7: Check Cluster Status

```bash
nodetool status
```

A healthy node normally shows `UN`:

* `U` = Up
* `N` = Normal

### Step 8: Test CQL Connectivity

```bash
cqlsh
```

This verifies connectivity to the ScyllaDB CQL interface.

# 6. Configuration

ScyllaDB configuration is primarily managed through configuration files under `/etc/scylla/` and `/etc/scylla.d/`.

The main configuration file is:

```text
/etc/scylla/scylla.yaml
```

Other important configuration files include:

```text
/etc/default/scylla-server
/etc/scylla/cassandra-rackdc.properties
/etc/scylla.d/io.conf
```

---

### Example Configuration

```yaml
cluster_name: 'Production-Scylla-Cluster'

seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
      - seeds: "10.0.1.10,10.0.1.11"

listen_address: 10.0.1.10
rpc_address: 0.0.0.0
```

> Configuration values must be adjusted according to the actual cluster network, node IP addresses, security requirements, and deployment topology.

---


# 7. Basic CQL Operations

ScyllaDB uses CQL (Cassandra Query Language) for database operations.

Common CQL operations include:

| **Operation**     | **Purpose**                         |
| ----------------- | ----------------------------------- |
| `CREATE KEYSPACE` | Create a logical database namespace |
| `CREATE TABLE`    | Create a table                      |
| `INSERT`          | Add data                            |
| `SELECT`          | Read data                           |
| `UPDATE`          | Modify data                         |
| `DELETE`          | Delete data                         |
| `DESCRIBE`        | Inspect schema information          |

# 8. Maintenance

Regular maintenance helps maintain ScyllaDB cluster performance, availability, and reliability.

| **Task**         | **Command / Action**                                   |
| ---------------- | ------------------------------------------------------ |
| Check service    | `sudo systemctl status scylla-server`                  |
| Start service    | `sudo systemctl start scylla-server`                   |
| Stop service     | `sudo systemctl stop scylla-server`                    |
| Restart service  | `sudo systemctl restart scylla-server`                 |
| Enable at boot   | `sudo systemctl enable scylla-server`                  |
| Check version    | `scylla --version`                                     |
| Check cluster    | `nodetool status`                                      |
| Check logs       | `journalctl -u scylla-server`                          |
| Check compaction | `nodetool compactionstats`                             |
| Check tables     | `nodetool tablestats`                                  |
| Backup           | Configure ScyllaDB backup procedures                   |
| Upgrade          | Follow the release-specific ScyllaDB upgrade procedure |

> Do not manually delete ScyllaDB database files or SSTables to resolve storage issues. Follow the appropriate ScyllaDB operational procedure.

---

# 9. Monitoring

Monitoring helps identify performance problems, resource exhaustion, node failures, and service availability issues.

| **Metric / Check** | **Purpose**                      | **Command / Tool**                    |
| ------------------ | -------------------------------- | ------------------------------------- |
| Node Status        | Detect failed or unhealthy nodes | `nodetool status`                     |
| CPU Usage          | Detect CPU saturation            | `top` / `htop`                        |
| Memory Usage       | Detect memory pressure           | `free -h` / `top`                     |
| Disk Usage         | Prevent storage exhaustion       | `df -h`                               |
| Disk I/O           | Identify storage bottlenecks     | `iostat`                              |
| Read Latency       | Measure read performance         | ScyllaDB Monitoring                   |
| Write Latency      | Measure write performance        | ScyllaDB Monitoring                   |
| Read Throughput    | Measure read workload            | ScyllaDB Monitoring                   |
| Write Throughput   | Measure write workload           | ScyllaDB Monitoring                   |
| Compaction         | Detect compaction pressure       | `nodetool compactionstats`            |
| SSTables           | Identify SSTable-related issues  | `nodetool tablestats`                 |
| Network Traffic    | Detect network saturation        | `ss -s` / `sar -n DEV`                |
| Service Status     | Verify service health            | `sudo systemctl status scylla-server` |
| CQL Port           | Verify port 9042                 | `ss -lntp \| grep 9042`               |
| Service Logs       | View recent logs                 | `journalctl -u scylla-server -n 100`  |
| Live Logs          | Monitor logs continuously        | `journalctl -u scylla-server -f`      |

---


# 10. Disaster Recovery

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

# 11. High Availability

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


---

# 12. Troubleshooting

Troubleshooting should begin by checking the service status, logs, cluster status, network connectivity, configuration, and available system resources.

| **Problem**                 | **First Checks**             | **Useful Command**               |
| --------------------------- | ---------------------------- | -------------------------------- |
| Service not starting        | Service, logs, configuration | `systemctl status scylla-server` |
| Node down                   | Service and network          | `nodetool status`                |
| CQL unavailable             | Port and RPC configuration   | `ss -lntp \| grep 9042`          |
| Node cannot join            | Cluster name, seeds, network | Check `scylla.yaml`              |
| High disk usage             | Disk and data directories    | `df -h`                          |
| High CPU                    | Processes and workload       | `top` / `htop`                   |
| High latency                | CPU, disk, network, queries  | Monitoring metrics               |
| Cluster communication issue | Firewall and ports           | `nc -zv <IP> 7000`               |

---


# 13. FAQs

### Is ScyllaDB Cassandra compatible?

Yes. ScyllaDB supports CQL and Cassandra-compatible clients and applications.

###  Does ScyllaDB support DynamoDB?

Yes. ScyllaDB provides the Alternator API, which provides DynamoDB-compatible access.


###  Does ScyllaDB support multiple datacenters?

Yes. ScyllaDB supports multi-datacenter deployments and topology-aware replication.

---

# 14. Contact Information

| Name |         Email Address             |
| ---- | ----------------------------------|
| Ritu | ritu.dogra.snaatak@mygurukulam.co |

---

# 15. References

| **Reference**                                                                                                           | **Purpose**                         |
| ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| [ScyllaDB Documentation](https://docs.scylladb.com/manual/stable/)                                                      | Official ScyllaDB documentation     |
| [ScyllaDB Installation](https://docs.scylladb.com/manual/stable/getting-started/install-scylla/)                        | ScyllaDB installation procedures    |
| [ScyllaDB System Requirements](https://docs.scylladb.com/manual/stable/getting-started/system-requirements.html)        | Hardware and platform requirements  |
| [ScyllaDB Architecture](https://docs.scylladb.com/manual/stable/architecture/)                                          | ScyllaDB architecture documentation |
| [ScyllaDB Features](https://docs.scylladb.com/manual/stable/features/)                                                  | ScyllaDB feature documentation      |
| [ScyllaDB GitHub](https://github.com/scylladb/scylladb)                                                                 | ScyllaDB source repository          |
| [OT-MICROSERVICES Software Template](https://github.com/OT-MICROSERVICES/documentation-template/wiki/Software-Template) | Reference documentation template    |

---
