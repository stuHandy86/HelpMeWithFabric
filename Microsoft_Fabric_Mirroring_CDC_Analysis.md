
# Microsoft Fabric Mirroring with SQL Server CDC: Risks & Benefits

## Overview

This document outlines the **benefits and risks** of enabling **Change Data Capture (CDC)** on SQL Server, particularly in the context of integrating with **Microsoft Fabric Mirroring**. It also addresses how this setup affects SQL Server load and Fabric capacity unit (CU) consumption.

---

## ✅ Benefits of Enabling CDC

| Category        | Benefit                             | Explanation                                                                 |
|----------------|--------------------------------------|-----------------------------------------------------------------------------|
| **Performance** | Low-impact change tracking           | CDC reads the transaction log asynchronously, without affecting DML performance. |
| **Efficiency**  | Supports incremental data sync       | Only new/changed/deleted rows are captured, reducing load compared to full refreshes. |
| **Compatibility** | Designed for analytics pipelines    | Ideal for modern data architectures like Microsoft Fabric mirroring and lakehouses. |
| **No schema change** | Non-invasive implementation     | No triggers or schema changes required — CDC operates via system tables. |
| **Historical tracking** | Change history available      | You can retain change history for days, enabling time-windowed ingestion or audit. |
| **Granular control** | Table-level activation           | Only enable CDC on necessary tables to minimize load and noise. |
| **Improved scalability** | Offload refresh logic        | Works well with multiple downstream consumers via mirroring — one source, many targets. |

---

## ⚠️ Risks and Challenges of Enabling CDC

| Category         | Risk                                   | Mitigation                                                                   |
|------------------|----------------------------------------|------------------------------------------------------------------------------|
| **Disk Usage**    | CDC tables can grow large              | Tune the **retention period** and enable regular cleanup jobs. Monitor CDC table sizes. |
| **Log Bloat**     | Delayed consumption causes log growth  | Ensure downstream systems (like Fabric) **consume changes frequently**.      |
| **Backup Size**   | Full/log backups increase due to CDC metadata | Adjust backup strategies and storage planning.                          |
| **Maintenance Overhead** | Requires job monitoring        | Automate job health checks and alerting via SQL Agent or monitoring tools.   |
| **Resource Usage** | Adds CPU/memory/I/O under high DML load | Monitor CDC impact via `sys.dm_cdc_*` views. Consider enabling only during low-load windows. |
| **Security**      | CDC metadata is accessible             | Restrict access to CDC schema using roles/permissions.                        |
| **Not suitable for all tables** | High-frequency update tables may be noisy | Evaluate each table’s suitability before enabling CDC.                 |
| **Version Limitations** | Limited to supported editions   | Ensure your edition supports CDC and is compatible with Fabric mirroring.    |

---

## 🔧 Performance Impact of CDC on SQL Server

- **Minimal impact on OLTP**: CDC reads the log asynchronously.
- **Some CPU/I/O overhead**: Due to writing to CDC change tables.
- **Increased disk usage**: Depending on volume of changes and retention settings.
- **Transaction log growth**: If changes are not consumed/cleaned up quickly.
- **Requires cleanup jobs**: To prevent table and log bloat.

---

## 🔍 Impact on Fabric Capacity Unit (CU) Consumption

### With Fabric Mirroring:
- **One-time ingestion** (initial sync) incurs higher CU usage.
- **Ongoing delta ingestion** is efficient and predictable.
- **All semantic models reuse the same mirrored data**, reducing total CU consumption vs. per-model refreshes.

### Without Mirroring:
- Multiple semantic models each refresh data directly from SQL Server.
- High CU consumption from repeated ingestion, transformation, and processing.
- Greater burden on SQL Server and Fabric compute.

---

## 📌 Best Practices for Enabling CDC

- Enable CDC **only on necessary tables**.
- Set **retention period** to match sync frequency (e.g., 24–48 hours).
- Monitor CDC table size and job status (`sys.sp_cdc_cleanup_change_table`, `sys.dm_cdc_*`).
- Secure access to `cdc` schema and change tables.
- Place CDC change tables on **fast storage**, if possible.
- Schedule **cleanup jobs** appropriately.

---

## 🎯 Summary Statement for IT Stakeholders

> "Enabling CDC allows Microsoft Fabric to ingest only changed data, reducing load on SQL Server and improving scalability. When configured properly, CDC introduces low, manageable overhead and helps centralize analytics in a modern, cloud-first architecture. Mirroring with CDC offloads Power BI refresh load, reduces CU consumption in Fabric, and enables real-time analytics without burdening transactional systems."

