
# Microsoft Fabric Mirroring with SQL Server CDC: Risks & Benefits

## Overview

This document outlines the **benefits, risks, and performance considerations** of enabling **Change Data Capture (CDC)** on SQL Server, particularly in the context of **Microsoft Fabric Mirroring**. It includes details on how CDC affects SQL Server load, how it optimizes Power BI refreshes, and how it impacts Capacity Unit (CU) consumption in Fabric.

---

## 📘 Official Documentation

- [Microsoft Fabric Mirroring Overview](https://learn.microsoft.com/en-us/fabric/data-engineering/mirroring-overview)
- [SQL Server CDC - Change Data Capture](https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server)
- [Monitor Capacity Unit Usage in Fabric](https://learn.microsoft.com/en-us/fabric/admin/capacity-metrics-app)

---

## ✅ Benefits of Enabling CDC

| Category        | Benefit                             | Explanation                                                                 |
|----------------|--------------------------------------|-----------------------------------------------------------------------------|
| **Performance** | Low-impact change tracking           | CDC reads the transaction log asynchronously, without impacting OLTP performance. |
| **Efficiency**  | Enables incremental data sync        | Only new/changed/deleted rows are captured, reducing load vs. full refreshes. |
| **Modern Integration** | Designed for analytics pipelines | Seamlessly integrates with Microsoft Fabric Mirroring to reduce refresh load. |
| **No Schema Change** | Non-invasive implementation     | No changes to application logic or schema — CDC works via system tables. |
| **Historical Tracking** | Change history available      | You can retain changes for a specific window (e.g., 24–72 hours). |
| **Granular Control** | Table-level activation           | Only activate CDC on required tables to minimize resource impact. |
| **Scalability** | Supports multiple consumers          | One mirrored stream can be reused by many semantic models and Power BI datasets. |

---

## ⚠️ Risks and Challenges of Enabling CDC

| Category         | Risk                                   | Mitigation                                                                   |
|------------------|----------------------------------------|------------------------------------------------------------------------------|
| **Disk Usage**    | CDC change tables can grow large       | Set appropriate retention (24–48 hrs) and monitor table sizes. |
| **Log Growth**    | Changes not consumed may bloat the transaction log | Ensure downstream consumers read frequently and run cleanup jobs. |
| **Backup Size**   | Larger backups due to CDC metadata     | Adjust backup plans to accommodate growth. |
| **Maintenance Overhead** | Job monitoring and management  | Use SQL Agent to automate and alert on CDC jobs. |
| **CPU/I/O Load**  | Extra load under high DML rates        | Monitor performance and isolate CDC tables on fast storage if possible. |
| **Security**      | Exposure of change data                | Restrict access to the `cdc` schema via roles and permissions. |
| **Version Limitations** | Not available on all editions    | CDC is supported in SQL Server Standard and Enterprise editions (not Express). |

---

## 🔧 Performance Impact of CDC on SQL Server

### 🔹 Minimal OLTP Impact
- CDC operates **asynchronously**, reading the transaction log in the background.
- **No triggers** are added; transactional workloads remain unaffected.

### 🔹 Additional Resource Usage
- **CPU**: Light-to-moderate depending on DML volume.
- **I/O**: Writes to CDC change tables; may increase with frequent changes.
- **Transaction Log**: Log must be read and retained until processed; requires frequent cleanup.
- **Storage**: Space needed for `cdc` tables is proportional to change volume and retention.

### 🔹 Cleanup Considerations
- CDC creates **capture and cleanup jobs**.
- If cleanup fails or lags, change tables and logs can grow significantly.

---

## 🔄 How Microsoft Fabric Mirroring Uses CDC

- Fabric Mirroring uses **change data capture** to **synchronize SQL Server tables** into OneLake incrementally.
- Once mirrored, Power BI and other workloads **read data from OneLake**, not from SQL Server.
- This **offloads the refresh and query load** from the source SQL Server, improving performance and scalability.

See: [Microsoft Fabric Mirroring Overview](https://learn.microsoft.com/en-us/fabric/data-engineering/mirroring-overview)

---

## ⚙️ CU (Capacity Unit) Impact in Fabric

### 📈 Without Mirroring
- Every Power BI dataset refresh pulls from the source SQL Server.
- Multiple refreshes daily (often 10s or 100s) add up to heavy CU and SQL Server load.
- Each refresh performs ingestion, transformation, model processing.

### ✅ With Mirroring
- **One initial sync** and **ongoing delta ingestion** from SQL Server into OneLake.
- Downstream models and semantic layers **read from OneLake**, not SQL Server.
- Much lower **CU cost per refresh**, and centralized ingestion improves efficiency.

📌 Learn more: [Monitor Capacity Usage in Fabric](https://learn.microsoft.com/en-us/fabric/admin/capacity-metrics-app)

---

## 📌 Best Practices When Using CDC with Fabric Mirroring

- 🔸 Enable CDC **only on tables needed** for analytics.
- 🔸 Set a **short retention period** (e.g. 1–2 days) if syncing frequently.
- 🔸 Monitor `cdc.lsn_time_mapping` and `cdc.<capture_instance>` table sizes.
- 🔸 Verify SQL Agent jobs (capture/cleanup) are **running reliably**.
- 🔸 Use **indexes on CDC tables** if querying them outside Fabric.
- 🔸 Apply **access control** to the `cdc` schema to protect sensitive change data.

---

## 🎯 Summary for IT Stakeholders

> Enabling CDC on SQL Server enables Microsoft Fabric to sync changes efficiently using Mirroring. This reduces load on transactional systems, avoids repeated refreshes, and centralizes analytics in a modern lakehouse architecture. While CDC introduces some manageable overhead, it is significantly more efficient and scalable than traditional full-extract or direct-query models.

---

## 📎 References

- [Change Data Capture (SQL Server)](https://learn.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server)
- [Microsoft Fabric Mirroring Overview](https://learn.microsoft.com/en-us/fabric/data-engineering/mirroring-overview)
- [Monitor Fabric Capacity](https://learn.microsoft.com/en-us/fabric/admin/capacity-metrics-app)
