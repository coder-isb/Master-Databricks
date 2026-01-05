# Chapter 13 – Disaster Recovery, High Availability & Multi-Region Architecture in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Platform Engineer | Cloud Architect | MLOps Engineer  
**Interview Focus:** DR strategies, Delta Lake recovery, multi-region architecture, fault-tolerant streaming, RTO/RPO planning, backup and retention policies  

---

## 1. Why DR & High Availability Matter

In production Lakehouse systems, **data loss, corruption, or downtime** can lead to significant business impact. Disaster recovery (DR) and high availability (HA) planning ensure:

- **Business continuity** during cloud or regional outages  
- **Rapid recovery** from accidental deletes or data corruption  
- Minimal data loss via **RPO (Recovery Point Objective)**  
- Fast system restoration via **RTO (Recovery Time Objective)**  

Interviews test your understanding of **Delta Lake resilience, cloud storage replication, and streaming fault tolerance**.

---

## 2. DR Strategies for Delta Lake

Delta Lake natively supports **ACID transactions, versioning, and checkpoints**, enabling several DR strategies:

1. **Time Travel:** Restore tables to previous versions or points in time  
2. **Delta Table Backups:** Copy Delta tables to separate storage accounts or regions  
3. **Cross-Region Replication:** Maintain copies of Delta tables across regions for HA  
4. **Continuous Streaming Checkpoints:** Ensure replayable data pipelines  

---

### Interview Questions & Answers

**Q1. How does Delta Lake support disaster recovery?**  

- Transaction logs (`_delta_log`) store metadata for every change.  
- Time Travel allows restoring table state to a previous version.  
- Checkpoints and versioned Parquet files provide consistency for backups.  

**Q2. Trade-offs: Frequent backups vs cost**  

Frequent backups reduce potential data loss but increase storage cost. Delta’s Time Travel reduces the need for full backups for short retention periods.

---

## 3. Time Travel Restoration

**Time Travel** enables restoring data using:

- **Version number**: `SELECT * FROM table VERSION AS OF 42`  
- **Timestamp**: `SELECT * FROM table TIMESTAMP AS OF '2026-01-01 10:00:00'`  

**Use cases:**

- Recover accidental deletes  
- Audit historical data  
- Re-run analytics on previous snapshots  

**Retention considerations:** Delta’s `delta.logRetentionDuration` controls how long versions are kept; beyond that, data must rely on backups.

---

### Interview Questions & Answers

**Q3. How do you recover a table after accidental deletes?**  

- Identify last correct version in `_delta_log`  
- Restore via `RESTORE TABLE` or `COPY INTO`  
- Optionally, vacuum older versions to reclaim storage  

**Q4. Trade-offs: Time Travel vs full backup restore**  

Time Travel is faster and convenient for short-term recovery. Full backups are essential for long-term DR or cross-region recovery.

---

## 4. Backup Patterns and Retention

**Backup strategies:**

- **Incremental backups:** Copy only changed Delta logs and Parquet files  
- **Full backups:** Periodically copy entire Delta table  
- **Retention policies:** Configure short retention for Time Travel and maintain long-term cold backups in cheaper storage  

**Tools:** Databricks Jobs, cloud storage replication, and Delta sharing for backups.

---

### Interview Questions & Answers

**Q5. How do you balance backup frequency and cost?**  

- Incremental backups daily, full backups weekly  
- Use cloud lifecycle management to move older backups to cold storage  
- Prioritize mission-critical tables for more frequent backups  

**Q6. Trade-offs: Incremental vs full backup**  

Incremental saves storage and reduces job time but requires chaining versions for restoration. Full backups are easier to restore but more expensive.

---

## 5. Cross-Region Replication

For global HA:

- Replicate Delta tables across regions using **cloud-native replication** (S3 replication, ADLS geo-replication)  
- Ensure **transaction log consistency** and apply **idempotent writes**  
- Consider **network bandwidth, storage cost, and replication lag**  

---

### Interview Questions & Answers

**Q7. How do you implement cross-region DR for Delta Lake?**  

- Use cloud storage replication  
- Maintain table and `_delta_log` consistency  
- Test recovery periodically to validate RTO/RPO  

**Q8. Trade-offs: Synchronous vs asynchronous replication**  

Synchronous replication ensures zero data loss but increases write latency. Asynchronous replication is faster but risks minimal data loss during outages.

---

## 6. Multi-Region Architectural Patterns

Common patterns for HA:

1. **Active-Passive:** Primary region active, secondary region replicated for DR  
2. **Active-Active:** Both regions serve reads; writes routed to primary with replication  
3. **Hybrid:** Hot data in primary, cold data replicated for analytics  

**Key considerations:**

- Network latency between regions  
- Cross-region costs for storage and compute  
- Cluster deployment strategy (multi-region clusters or separate regional clusters)  

---

### Interview Questions & Answers

**Q9. How do you design a multi-region Delta Lake architecture?**  

- Choose active-active or active-passive based on SLA  
- Use Delta replication and cloud-native storage replication  
- Implement orchestration to failover pipelines automatically  

**Q10. Trade-offs: Active-active vs active-passive**  

Active-active improves availability and load balancing but increases complexity and cost. Active-passive is simpler but longer failover time.

---

## 7. RTO/RPO Planning

- **RTO (Recovery Time Objective):** Maximum tolerable downtime  
- **RPO (Recovery Point Objective):** Maximum tolerable data loss  

Databricks design choices impact RTO/RPO:

- Delta Time Travel and checkpoints reduce RTO  
- Frequent replication and backups reduce RPO  
- Streaming checkpoints ensure recoverable pipeline states  

---

### Interview Questions & Answers

**Q11. How do you minimize RPO in Delta Lake pipelines?**  

- Use real-time streaming with checkpointing  
- Replicate critical tables across regions  
- Automate frequent backups for mission-critical tables  

**Q12. Trade-offs: Aggressive RPO/RTO vs cost**  

Aggressive RPO/RTO reduces data loss and downtime but increases infrastructure and storage costs. Balance according to business SLA.

---

## 8. Fault-Tolerant Streaming Design

**Best practices for streaming HA:**

- Use **structured streaming with checkpointing**  
- Enable **write-ahead logs (WAL)** for exactly-once semantics  
- Partition streaming data efficiently to reduce task failures  
- Combine **retry policies** and **event deduplication**  

---

### Interview Questions & Answers

**Q13. How do you recover a streaming pipeline after failure?**  

- Restart from the last checkpoint  
- Reprocess data within watermark limits  
- Deduplicate late-arriving data to avoid double-counting  

**Q14. Trade-offs: Frequent checkpointing vs throughput**  

Frequent checkpointing improves recoverability but adds I/O overhead. Infrequent checkpointing reduces cost but increases data loss risk.

---

## 9. Recovering Accidental Deletes and Corrupt Files

- Use **Time Travel** to restore deleted records or tables  
- Validate data integrity with **checksums or Delta Table validation**  
- Vacuum cautiously—deleted files may not be recoverable if removed from storage  

---

### Interview Questions & Answers

**Q15. How do you recover corrupt Delta files?**  

- Identify corrupt Parquet files in `_delta_log`  
- Restore from backups or previous Time Travel versions  
- Run **OPTIMIZE** to compact and revalidate remaining data  

**Q16. Trade-offs: Immediate restore vs waiting for batch recovery**  

Immediate restore minimizes downtime but may consume compute for large tables. Batch recovery is cheaper but increases RTO.

---

## 10. Production Scenarios

### Scenario 1: Accidental Table Delete

A production Delta table is accidentally dropped. Use Time Travel or backup to restore the table to its previous state with minimal data loss.

### Scenario 2: Regional Cloud Outage

Primary region fails. Failover to secondary region with replicated Delta tables and clusters. Streaming pipelines restart from last checkpoints.

### Scenario 3: Streaming Data Corruption

A streaming pipeline writes malformed records. Detect corrupt files, pause ingestion, restore last good version, and resume streaming with deduplication.

---

## 11. Best Practices & Common Pitfalls

### Best Practices
- Enable Delta Time Travel for short-term DR  
- Implement incremental or full backups for long-term DR  
- Use cross-region replication for mission-critical tables  
- Monitor checkpoints for streaming pipelines  
- Plan RTO/RPO based on business requirements  
- Validate recovery process regularly  

### Common Pitfalls
- Ignoring `_delta_log` version retention policies  
- Over-relying on Time Travel without backups  
- Not testing cross-region failover  
- Skipping checkpoint and watermark monitoring in streaming pipelines  
- Vacuuming too aggressively, removing necessary historical versions  

---

## Key Takeaways

- Delta Lake provides **native mechanisms for disaster recovery** through Time Travel, checkpoints, and transaction logs  
- Multi-region replication and HA clusters ensure **business continuity** during cloud outages  
- Streaming pipelines require checkpointing and exactly-once semantics for fault tolerance  
- RTO and RPO planning balances cost and recovery requirements  
- Regular testing, monitoring, and backup validation are critical for operational excellence  
