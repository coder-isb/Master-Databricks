# Chapter 15 – Scenario-Based System Design & Architecture Problems in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Solution Architect | Platform Engineer | Analytics Engineer  
**Interview Focus:** End-to-end Lakehouse design, multi-team workload management, CDC pipelines, batch + streaming unification, cost-efficient architecture, governance, SCD modeling, and high-throughput ingestion  

---

## 1. Why System Design Scenarios Matter

In interviews for top product companies or GCCs, **system design problems test both technical knowledge and architecture thinking**. Databricks candidates are expected to design:

- Scalable Lakehouse pipelines (Bronze/Silver/Gold)  
- Incremental ingestion strategies using CDC or Autoloader  
- Real-time and batch processing integration  
- Cost-optimized, secure, and governable architectures  

Interviewers focus on **trade-offs, operational considerations, and maintainability**.

---

## 2. End-to-End Lakehouse Design

A complete Lakehouse architecture often involves:

1. **Bronze Layer:** Raw data ingestion (batch/streaming)  
2. **Silver Layer:** Cleansed, standardized, and enriched tables  
3. **Gold Layer:** Aggregated, analytics-ready datasets  

**Key considerations:**

- Data partitioning and Z-Ordering for performance  
- Schema evolution handling  
- Caching or pre-aggregation for downstream BI/ML workloads  

---

### Interview Questions & Answers

**Q1. How would you design an end-to-end Lakehouse for a retail company?**  

- Bronze: ingest POS, web logs, and ERP data (Autoloader, streaming)  
- Silver: clean, standardize, deduplicate, handle schema drift  
- Gold: aggregated metrics for dashboards, ML features, and partner reporting  
- Implement Delta Lake features like Time Travel, ACID transactions, and CDC  

**Q2. Trade-offs: Bronze retention vs storage cost**  

Longer retention allows replay and historical audits but increases storage costs. Shorter retention saves cost but limits time-travel use.

---

## 3. Multi-Team Workload Architecture

**Challenges:**

- Multiple teams sharing the same Lakehouse  
- Varying SLA and performance requirements  
- Avoiding resource contention  

**Strategies:**

- High-concurrency clusters for BI  
- Dedicated compute clusters for ML workloads  
- Job scheduling and orchestration via Databricks Workflows  
- Tagging and cost governance policies  

---

### Interview Questions & Answers

**Q3. How do you manage multi-team workloads efficiently?**  

- Separate workloads by clusters or pools  
- Use instance pools to reduce cluster startup latency  
- Prioritize SLAs via job scheduling and autoscaling  

**Q4. Trade-offs: Shared vs dedicated clusters**  

Shared clusters save cost but may lead to resource contention. Dedicated clusters improve performance but increase operational cost.

---

## 4. CDC and Incremental Pipelines

**Change Data Capture (CDC)** is critical for incremental ingestion:

- Use **Autoloader cloudFiles** or streaming sources for real-time CDC  
- Maintain metadata for incremental processing  
- Combine with Delta Lake Merge operations for upserts  

---

### Interview Questions & Answers

**Q5. How do you implement CDC for multiple source systems?**  

- Extract CDC logs (binlog, change tables)  
- Stage raw changes in Bronze  
- Merge into Silver/Gold layers using Delta Merge or UPSERT operations  

**Q6. Trade-offs: Micro-batch vs streaming CDC**  

Streaming provides near-real-time updates but adds complexity. Micro-batch is simpler but introduces latency.

---

## 5. Migration from On-Prem Systems

**Key steps for migrating legacy systems to Databricks:**

- Extract raw data from relational databases or file systems  
- Stage in cloud object storage (S3, ADLS, GCS)  
- Incrementally transform into Delta Lake tables  
- Validate historical data for accuracy  

---

### Interview Questions & Answers

**Q7. How would you migrate an on-prem data warehouse to Databricks?**  

- Stage raw tables in Bronze (full load initially)  
- Apply incremental CDC pipelines to Silver  
- Use Delta Lake features to maintain history and enable analytics  

**Q8. Trade-offs: Lift-and-shift vs full redesign**  

Lift-and-shift is faster but may not leverage Lakehouse optimizations. Full redesign is optimal but time-consuming.

---

## 6. Handling Schema Evolution at Scale

**Challenges:**

- Frequent changes in source systems  
- Avoiding pipeline failures due to schema drift  
- Maintaining consistent downstream analytics  

**Strategies:**

- Use **Delta Lake Auto Merge Schema**  
- Validate schema in Silver layer before Gold aggregation  
- Versioned schemas for rollback and auditing  

---

### Interview Questions & Answers

**Q9. How do you handle schema evolution in production?**  

- Enable Delta Merge Schema or Auto Loader schema evolution  
- Maintain metadata registry for tracking schema changes  
- Implement alerts for incompatible changes  

**Q10. Trade-offs: Strict vs flexible schema enforcement**  

Strict schema reduces downstream errors but may break pipelines on source changes. Flexible schema increases pipeline resilience but may allow inconsistent data types.

---

## 7. Combining Batch & Streaming Pipelines

**Unified pipelines** reduce duplication:

- Use **Structured Streaming + Autoloader** for micro-batch ingestion  
- Combine with batch pipelines for historical data  
- Maintain **single source of truth** in Delta Lake  

---

### Interview Questions & Answers

**Q11. How would you combine batch and streaming ETL?**  

- Load historical batch data into Bronze  
- Stream new data with checkpointing  
- Merge into Silver/Gold tables  
- Ensure idempotent merges to avoid duplicates  

**Q12. Trade-offs: Unified vs separate pipelines**  

Unified pipelines simplify architecture but require robust handling of late-arriving or reprocessed data. Separate pipelines are simpler but can duplicate work.

---

## 8. Cost-Efficient Architecture Design

Strategies:

- Use spot/preemptible nodes for non-critical workloads  
- Enable autoscaling and auto-termination  
- Pre-aggregate Gold tables to reduce BI query cost  
- Use instance pools to reduce cluster startup latency  

---

### Interview Questions & Answers

**Q13. How do you design a cost-efficient Lakehouse?**  

- Use multi-tiered Bronze/Silver/Gold architecture  
- Optimize partitioning, caching, and Z-ordering  
- Mix on-demand and spot clusters based on SLA  

**Q14. Trade-offs: Cost vs performance**  

Optimizing for cost may slightly increase query latency. Prioritizing performance may increase compute and storage costs.

---

## 9. Governance and Security for Enterprises

Enterprise architectures must consider:

- Unity Catalog for centralized governance  
- Row-level and column-level security  
- Audit logging and compliance  
- Secrets management for external integrations  

---

### Interview Questions & Answers

**Q15. How do you enforce security across multi-team pipelines?**  

- Implement RBAC via Unity Catalog  
- Restrict access to sensitive columns or rows  
- Enable audit logs and monitor cluster usage  

**Q16. Trade-offs: Centralized vs distributed governance**  

Centralized governance ensures consistency but may slow team agility. Distributed governance is flexible but risks inconsistencies.

---

## 10. SCD1, SCD2, CDC Modeling

- **SCD1 (overwrite)**: Simple update of records  
- **SCD2 (history)**: Maintain historical records with effective dates  
- **CDC pipelines**: Merge incremental changes into SCD tables  

---

### Interview Questions & Answers

**Q17. How do you implement SCD2 with Delta Lake?**  

- Use Delta Merge to insert new rows with history  
- Track `start_date` and `end_date` columns  
- Ensure deduplication and idempotency  

**Q18. Trade-offs: SCD1 vs SCD2**  

SCD1 is simpler and faster but loses history. SCD2 preserves history but increases storage and merge complexity.

---

## 11. High-Throughput Ingestion Architecture

Key considerations:

- Partitioning data in Bronze layer to optimize writes  
- Use Autoloader for incremental ingestion  
- Batch or micro-batch writes for high ingestion rates  
- Monitor streaming lag and backpressure  

---

### Interview Questions & Answers

**Q19. How would you design high-throughput ingestion?**  

- Parallel ingestion streams using Autoloader  
- Partitioning and file compaction in Delta Lake  
- Autoscaling clusters to handle peak ingestion  

**Q20. Trade-offs: High concurrency vs single-thread ingestion**  

High concurrency reduces latency but may increase shuffle overhead and cost. Single-thread ingestion is simple but slower.

---

## 12. Production Scenarios

### Scenario 1: Multi-Team Retail Lakehouse

Bronze ingests POS, web, and inventory data. Silver cleans and standardizes. Gold aggregates metrics for finance, sales, and ML features. Teams use dedicated clusters for SLA isolation.

### Scenario 2: Hybrid Streaming + Batch Pipeline

Streaming IoT sensor data ingested via Event Hubs. Historical batch data loaded daily. Merge ensures single source of truth for ML predictions.

### Scenario 3: Enterprise Migration

On-prem Oracle data migrated to Delta Lake. CDC implemented via Oracle redo logs. Governance enforced via Unity Catalog. SCD2 tables maintain historical records.

---

## 13. Best Practices & Common Pitfalls

### Best Practices
- Plan end-to-end Lakehouse with Bronze/Silver/Gold layers  
- Use Delta Merge for CDC and incremental updates  
- Enable schema evolution handling  
- Combine batch + streaming pipelines carefully  
- Optimize partitioning, Z-Ordering, and caching  
- Enforce security and governance with Unity Catalog  
- Monitor ingestion pipelines for lag and throughput  

### Common Pitfalls
- Ignoring SLA differences between multi-team workloads  
- Overcomplicating pipelines without incremental processing  
- Neglecting schema evolution or late-arriving data  
- Over-reliance on full-table updates instead of CDC/SCD2  
- Missing governance and audit controls  

---

## Key Takeaways

- Scenario-based system design tests both architecture thinking and Databricks technical expertise  
- Bronze/Silver/Gold layers are foundational for scalable Lakehouse pipelines  
- CDC, SCD, and incremental ETL ensure efficient updates and historical tracking  
- Combining batch and streaming pipelines reduces duplication but requires careful design  
- Cost, performance, and governance trade-offs are central to enterprise-grade architectures  

