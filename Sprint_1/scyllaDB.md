# ScyllaDB Documentation

<img width="128" height="128" alt="ScyllaDB Icon" src="https://github.com/user-attachments/assets/3aee4f01-f9f5-49ea-b288-a6d4004d456c" />

---
# Author
| **Author**    | **Created On** | **Version** | **Last Updated By** | **Last Edited On** | **L0 Reviewer** | **L1 Reviewer** | **L2 Reviewer** |
| ------------- | -------------- | ----------- | ------------------- | ------------------ | --------------- | --------------- | --------------- |
| Vashishtha Prakash | 07-09-2026     | 1         | Vashishtha Prakash       | 07-09-2026         | Sunny/Shubham   | Shreya / Nikita  | Piyush Upadhyay   |


---

# 1. Introduction

This documentation provides a clear and structured guide for installing, configuring, managing, monitoring, and troubleshooting ScyllaDB. It helps administrators and developers understand the software requirements, features, commands, dependencies, security, high availability, backup, and disaster recovery procedures required for reliable ScyllaDB operations.

---

# 2. Purpose

ScyllaDB is designed for high-performance applications that require low-latency data access, high throughput, and horizontal scalability. It is suitable for real-time applications, IoT, time-series data, messaging, gaming, analytics, and recommendation systems. It can also support modern AI/ML workloads, including applications that require vector search and large-scale distributed data processing.

---

# 3. Features

| **Feature** | **Description** |
|---|---|
| Distributed Architecture | Data is distributed across multiple nodes. |
| High Performance | Uses the Seastar framework and shard-per-core architecture. |
| Horizontal Scaling | Capacity can be increased by adding nodes. |
| Cassandra Compatibility | Supports CQL and Cassandra-compatible clients. |
| DynamoDB Compatibility | Provides the Alternator API. |
| Replication | Replicates data across nodes and datacenters. |
| Fault Tolerance | Supports continued operation during node failures. |
| Configurable Consistency | Supports multiple consistency levels. |
| Change Data Capture | Captures database changes for downstream processing. |
| Vector Search | Supports similarity search using vector embeddings. |
| Full-Text Search | Supports text-based search workloads. |
| Lightweight Transactions | Supports conditional operations with stronger consistency. |
| Backup & Restore | Provides database backup and recovery capabilities. |

---

# 4. Software Overview

ScyllaDB is a distributed NoSQL wide-column database that uses CQL as its primary query language and provides Cassandra compatibility. It also supports the DynamoDB-compatible Alternator API and runs on x86_64 and AArch64 architectures. The main ScyllaDB service is managed through `scylla-server` on Linux systems. Its primary configuration file is `/etc/scylla/scylla.yaml`, while `cqlsh` and `nodetool` are commonly used for administration. The `scylla_setup` utility is used to configure system, storage, and ScyllaDB settings during initial setup.

---

# 5. Prerequisites

| **Requirement** | **Details** |
|---|---|
| OS | Supported Linux distribution |
| Architecture | x86_64 / AArch64 |
| CPU | Modern multi-core CPU |
| RAM | Workload dependent |
| Storage | SSD recommended |
| Filesystem | XFS recommended for production |
| Network | High-bandwidth network |
| Access | Root/sudo privileges |
| Connectivity | Required for installation and cluster communication |

---

# 6. System Requirements

| **Resource** | **Recommendation / Requirement** |
|---|---|
| CPU | Modern CPU with SSE4.2 support |
| CPU Cores | Workload dependent; 20–60 logical cores is a typical medium/high-workload guidance range |
| RAM | 16 GB or 2 GB per logical core, whichever is higher |
| Medium/High Workload RAM | Approximately 64–256 GB depending on workload |
| Storage | SSD strongly recommended |
| Disk/RAM Ratio | Approximately 30:1 as a general sizing guideline |
| Filesystem | XFS |
| Network | 10 Gbps or higher for large nodes |

> Exact production sizing must be based on workload, dataset size, replication, throughput, and latency requirements.

---

# 7. Ports

| **Port** | **Protocol** | **Purpose** | **Access** |
|---:|---|---|---|
| 22 | TCP | SSH administration | Admin network |
| 7000 | TCP | Node-to-node communication | Cluster nodes |
| 7001 | TCP | TLS node-to-node communication | Cluster nodes |
| 9042 | TCP | CQL client connections | Application network |
| 9142 | TCP | TLS CQL connections | Application network |
| 10000 | TCP | REST/management access | Trusted network |
| 7199 | TCP | JMX compatibility/management | Trusted network |

> Do not expose internal ScyllaDB ports directly to the public internet.

---

# 8. Dependencies

## 8.1 Runtime Dependencies

| **Dependency** | **Purpose** |
|---|---|
| Linux | Operating system |
| XFS | Production filesystem |
| systemd | Service management |
| TCP/IP | Client and cluster communication |
| CQL Driver | Application connectivity |

## 8.2 Administration Tools

| **Tool** | **Purpose** |
|---|---|
| `cqlsh` | Execute CQL commands |
| `nodetool` | Node and cluster administration |
| `scylla_setup` | System and storage configuration |
| ScyllaDB Manager | Cluster management |
| Monitoring Stack | Metrics and alerting |

---

# 9. Installation

| **Step** | **Command / Action** | **Purpose / Expected Result** |
|---|---|---|
| Update System | `sudo apt update && sudo apt upgrade -y` | Update the operating system. |
| Install ScyllaDB | Install ScyllaDB using the official installer/package method. | Install ScyllaDB on the system. |
| Verify Installation | `scylla --version` | Verify the installed ScyllaDB version. |
| Configure System | `sudo scylla_setup` | Configure system, storage, and ScyllaDB settings. |
| Start Service | `sudo systemctl start scylla-server` | Start the ScyllaDB service. |
| Enable Service | `sudo systemctl enable scylla-server` | Enable ScyllaDB to start automatically at boot. |
| Check Service | `sudo systemctl status scylla-server` | Verify that the service is running. |
| Check Cluster | `nodetool status` | Verify cluster and node health. A healthy node shows `UN` (Up and Normal). |
| Test CQL | `cqlsh` | Test CQL connectivity and open the CQL shell. |


---

# 10. Configuration

| **Configuration** | **Location / Example** |
|---|---|
| Main Configuration | `/etc/scylla/scylla.yaml` |
| Service Defaults | `/etc/default/scylla-server` |
| Datacenter/Rack | `/etc/scylla/cassandra-rackdc.properties` |
| I/O Configuration | `/etc/scylla.d/io.conf` |

## 10.1 Important Configuration Parameters

| **Parameter** | **Purpose** |
|---|---|
| `cluster_name` | Identifies the cluster |
| `seed_provider` | Provides initial cluster contact points |
| `listen_address` | Node-to-node communication address |
| `rpc_address` | Client communication address |
| `broadcast_address` | Address advertised to other nodes |
| `endpoint_snitch` | Determines topology awareness |

### Example

```yaml
cluster_name: 'Production-Scylla-Cluster'

seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
      - seeds: "10.0.1.10,10.0.1.11"

listen_address: 10.0.1.10
rpc_address: 0.0.0.0
```

---

# 11. Datacenter and Rack Configuration

| **Parameter** | **Example** |
|---|---|
| Datacenter | `dc1` |
| Rack | `rack1` |
| Configuration File | `/etc/scylla/cassandra-rackdc.properties` |

```properties
dc=dc1
rack=rack1
```

Datacenter and rack information is used for topology-aware replication and fault tolerance.

---

# 12. Basic CQL Operations

| **Operation** | **Purpose** |
|---|---|
| `CREATE KEYSPACE` | Create a logical database namespace |
| `CREATE TABLE` | Create a table |
| `INSERT` | Add data |
| `SELECT` | Read data |
| `UPDATE` | Modify data |
| `DELETE` | Delete data |
| `DESCRIBE` | Inspect schema information |

## 12.1 Create Keyspace

```sql
CREATE KEYSPACE ecommerce
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3
};
```

## 12.2 Create Table

```sql
USE ecommerce;

CREATE TABLE orders (
    customer_id uuid,
    order_id uuid,
    order_date timestamp,
    amount decimal,
    status text,
    PRIMARY KEY (customer_id, order_id)
);
```

## 12.3 Insert Data

```sql
INSERT INTO orders (
    customer_id,
    order_id,
    order_date,
    amount,
    status
)
VALUES (
    123e4567-e89b-12d3-a456-426614174000,
    987e6543-e21b-34d3-b654-426614174111,
    '2026-09-08',
    2499.00,
    'CONFIRMED'
);
```

## 12.4 Query Data

```sql
SELECT *
FROM orders
WHERE customer_id =
123e4567-e89b-12d3-a456-426614174000;
```

---

# 13. Maintenance

| **Task** | **Command / Action** |
|---|---|
| Check service | `systemctl status scylla-server` |
| Start service | `systemctl start scylla-server` |
| Stop service | `systemctl stop scylla-server` |
| Restart service | `systemctl restart scylla-server` |
| Enable at boot | `systemctl enable scylla-server` |
| Check version | `scylla --version` |
| Check cluster | `nodetool status` |
| Check logs | `journalctl -u scylla-server` |
| Backup | Configure ScyllaDB backup/restore |
| Upgrade | Follow release-specific upgrade procedure |

> Do not manually delete database files to solve storage problems.

---

# 14. Monitoring


| **Metric / Check** | **Purpose** | **Command** |
|---|---|---|
| Node Status | Detect failed or unhealthy nodes. | `nodetool status` |
| CPU Usage | Detect CPU saturation. | `top` / `htop` |
| Memory Usage | Detect memory pressure. | `free -h` / `top` |
| Disk Usage | Prevent storage exhaustion. | `df -h` |
| Disk I/O | Identify storage bottlenecks. | `iostat` |
| Read Latency | Measure read performance. | ScyllaDB Monitoring |
| Write Latency | Measure write performance. | ScyllaDB Monitoring |
| Read Throughput | Measure read workload. | ScyllaDB Monitoring |
| Write Throughput | Measure write workload. | ScyllaDB Monitoring |
| Compaction | Detect compaction pressure. | `nodetool compactionstats` |
| SSTables | Identify excessive SSTables. | `nodetool tablestats` |
| Network Traffic | Detect network saturation. | `ss -s` / `sar -n DEV` |
| Service Status | Verify ScyllaDB service health. | `sudo systemctl status scylla-server` |
| CQL Port | Verify that port `9042` is listening. | `ss -lntp \| grep 9042` |
| Service Logs | View recent ScyllaDB log entries. | `journalctl -u scylla-server -n 100` |
| Live Logs | Monitor ScyllaDB logs in real time. | `journalctl -u scylla-server -f` |
---

# 15. Backup

| **Backup Component** | **Purpose** |
|---|---|
| Database Backup | Protect database data |
| Object Storage | Store backups durably |
| Separate Region | Protect against regional failure |
| Backup Schedule | Meet RPO requirements |
| Restore Test | Verify backup usability |

Possible object-storage destinations:

| **Platform** | **Storage** |
|---|---|
| AWS | S3 |
| Google Cloud | Cloud Storage |
| Azure | Blob Storage |

---

# 16. Disaster Recovery

| **Failure Scenario** | **Protection Mechanism** |
|---|---|
| Node Failure | Replication |
| Disk Failure | Replicated data + backup |
| Data Corruption | Backup and restore |
| Accidental Deletion | Backup |
| AZ Failure | Multi-AZ deployment |
| Datacenter Failure | Multi-DC deployment |
| Region Failure | Cross-region strategy and backups |

## 16.1 Recommended DR Design

<img width="533" height="361" alt="image" src="https://github.com/user-attachments/assets/f743bc3a-db5e-44ac-8385-5768bf5f7b08" />


---

# 17. High Availability

| **HA Component** | **Recommendation** |
|---|---|
| Cluster | Use multiple nodes |
| Replication | Use an appropriate replication factor |
| Availability Zones | Distribute nodes across AZs |
| Datacenters | Use multiple DCs where required |
| Consistency | Select according to application needs |
| Backups | Maintain independent backups |
| Monitoring | Configure health monitoring and alerts |

### Example

<img width="668" height="331" alt="image" src="https://github.com/user-attachments/assets/55949030-7254-4e2e-8809-5659cd381241" />


---

# 18. Consistency and Replication

| **Concept** | **Description** |
|---|---|
| Replication Factor | Number of data replicas |
| Consistency Level | Required acknowledgement level |
| `ONE` | One replica acknowledgement |
| `QUORUM` | Majority of replicas |
| `LOCAL_QUORUM` | Quorum within local datacenter |
| `ALL` | All required replicas |
| NetworkTopologyStrategy | Topology-aware replication strategy |

Example:

```sql
CREATE KEYSPACE ecommerce
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3
};
```

---

# 19. Troubleshooting

| **Problem** | **First Checks** | **Useful Command** |
|---|---|---|
| Service not starting | Service, logs, configuration | `systemctl status scylla-server` |
| Node down | Service and network | `nodetool status` |
| CQL unavailable | Port and RPC configuration | `ss -lntp \| grep 9042` |
| Node cannot join | Cluster name, seeds, network | Check `scylla.yaml` |
| High disk usage | Disk and data directories | `df -h` |
| High CPU | Processes and workload | `top` / `htop` |
| High latency | CPU, disk, network, queries | Monitoring metrics |
| Cluster communication issue | Firewall and ports | `nc -zv <IP> 7000` |

## 19.1 Service Failure

```bash
sudo systemctl status scylla-server
journalctl -u scylla-server -n 100
```

Check:

```text
/etc/scylla/scylla.yaml
/etc/scylla.d/io.conf
```

## 19.2 Node Failure

```bash
nodetool status
```

Then verify:

```bash
sudo systemctl status scylla-server
```

## 19.3 CQL Connection Failure

```bash
cqlsh <IP_ADDRESS> 9042
```

Check:

```bash
ss -lntp | grep 9042
```

Verify:

- `rpc_address`
- Firewall
- Security Group
- Network ACL
- TLS configuration

## 19.4 High Disk Usage

```bash
df -h
```

```bash
du -sh /var/lib/scylla/*
```

Check for:

- Large datasets
- SSTables
- Compaction
- Backups
- Insufficient capacity

---

# 20. Security

| **Security Area** | **Recommendation** |
|---|---|
| Network | Restrict database ports to trusted networks |
| SSH | Allow only administrative sources |
| CQL | Use authentication and encryption where required |
| TLS | Enable TLS for sensitive environments |
| Firewall | Restrict cluster ports |
| Credentials | Store secrets securely |
| Backups | Encrypt and restrict backup access |
| Access Control | Apply least privilege |
| Updates | Keep ScyllaDB and OS updated |
| Monitoring | Alert on security and availability events |

---

# 21. Common Commands

| **Command** | **Purpose** |
|---|---|
| `scylla --version` | Show ScyllaDB version |
| `scylla_setup` | Configure system |
| `cqlsh` | Open CQL shell |
| `nodetool status` | Show cluster status |
| `nodetool info` | Show node information |
| `nodetool describecluster` | Show cluster information |
| `systemctl status scylla-server` | Check service |
| `systemctl restart scylla-server` | Restart service |
| `journalctl -u scylla-server` | View service logs |
| `ss -lntp` | Check listening ports |

---

# 22. Production Architecture

<img width="584" height="364" alt="image" src="https://github.com/user-attachments/assets/103e5c12-9283-4f35-a462-a33d5bc693d6" />

---

# 23. FAQs

| **Question** | **Answer** |
|---|---|
| Is ScyllaDB SQL? | No. It is a NoSQL wide-column database using CQL. |
| Is ScyllaDB Cassandra compatible? | Yes, it supports CQL and Cassandra-compatible clients. |
| Does ScyllaDB support DynamoDB? | Yes, through the Alternator API. |
| Does ScyllaDB support replication? | Yes. |
| Does ScyllaDB support multi-DC? | Yes. |
| Does ScyllaDB support AWS? | Yes. |
| Does ScyllaDB support Graviton? | Yes, supported AArch64/Graviton deployments are available. |
| Does ScyllaDB support Docker? | Yes. |
| Does ScyllaDB support vector search? | Yes. |
| Does ScyllaDB support CDC? | Yes. |
| What checks cluster health? | `nodetool status`. |
| What is the main configuration file? | `/etc/scylla/scylla.yaml`. |
| What does `UN` mean? | Up and Normal. |

---

# 24. Contact Information

| **Name** | **Email** |
|---|---|
| Vashishtha Prakash | vashishtha.prakash.snaatak@mygurukulam.co |

---

# 25. References

| **Reference** | **Purpose** |
|---|---|
| [ScyllaDB Documentation](https://docs.scylladb.com/manual/stable/) | Official documentation |
| [ScyllaDB Installation](https://docs.scylladb.com/manual/stable/getting-started/install-scylla/) | Installation procedures |
| [ScyllaDB System Requirements](https://docs.scylladb.com/manual/stable/getting-started/system-requirements.html) | Hardware and platform requirements |
| [ScyllaDB Architecture](https://docs.scylladb.com/manual/stable/architecture/) | Architecture documentation |
| [ScyllaDB Features](https://docs.scylladb.com/manual/stable/features/) | Feature documentation |
| [ScyllaDB GitHub](https://github.com/scylladb/scylladb) | Source repository |
| [OT-MICROSERVICES Software Template](https://github.com/OT-MICROSERVICES/documentation-template/wiki/Software-Template) | Documentation template |
