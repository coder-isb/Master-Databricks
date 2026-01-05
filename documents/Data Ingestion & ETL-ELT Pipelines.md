
# Chapter 5 – Data Ingestion & ETL / ELT Pipelines in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Senior Data Engineer | Platform Engineer  
**Interview Focus:** Ingestion Architecture, Auto Loader, Streaming vs Batch, Multi-Hop Pipelines, Incremental ETL, Schema Evolution, DLT, and Orchestration  

---

## 1. Why Data Ingestion & Pipeline Design Matter in Interviews

In real-world data platforms, ingestion and pipeline design determine whether data systems are **reliable, scalable, and maintainable**. Interviewers use these topics to assess if a candidate can move beyond writing Spark code and instead design **production-grade data flows**.

Poor ingestion design leads to:
- Data loss or duplication
- Inconsistent schemas
- Broken downstream analytics
- Expensive reprocessing
- Difficult recovery after failures

Strong candidates demonstrate understanding of **incremental processing, fault tolerance, schema evolution, and orchestration**, not just transformations.

---

## 2. Data Ingestion Architecture in the Lakehouse

Databricks ingestion architecture typically follows an **event-driven or file-driven model** where data lands in cloud object storage and is processed incrementally using Spark and Delta Lake.

Key principles include:
- Treating raw data as immutable
- Designing idempotent ingestion
- Separating ingestion from transformation
- Supporting both batch and streaming sources

This architecture enables unified batch and streaming processing while keeping data lineage clear.

---

## 3. Auto Loader (cloudFiles) – Ingestion Architecture

Auto Loader is Databricks’ optimized file ingestion mechanism designed for **incremental, scalable ingestion from cloud storage**.

Internally, Auto Loader works by:
- Tracking new files using metadata
- Avoiding directory listing bottlenecks
- Processing files incrementally with exactly-once semantics
- Integrating tightly with Structured Streaming

It supports two main modes:
- **Directory listing mode** (simpler, less scalable)
- **File notification mode** (event-driven, highly scalable)

### Interview Questions & Answers

**Q1. Why is Auto Loader preferred over manual file ingestion?**  

Auto Loader is designed to scale to millions of files without overwhelming cloud storage APIs. It provides built-in state management, fault tolerance, and exactly-once ingestion semantics, which are difficult to implement reliably with custom logic.

**Q2. How does Auto Loader achieve incremental ingestion?**  

Auto Loader maintains a checkpoint that tracks processed files. On restart or failure, it resumes from the last committed state, ensuring no data is reprocessed or missed.

**Q3. Trade-off: Directory listing vs file notification mode?**  

Directory listing is simpler to set up but does not scale well with large numbers of files. File notification mode is more complex to configure but provides near real-time ingestion and better scalability.

---

## 4. Batch Ingestion vs Micro-Batch Streaming

Databricks supports both traditional batch ingestion and micro-batch streaming using the same Spark APIs.

Batch ingestion processes a fixed snapshot of data, while micro-batch streaming continuously processes new data in small batches.

| Aspect | Batch Ingestion | Micro-Batch Streaming |
|------|-----------------|-----------------------|
| Latency | High | Low |
| Complexity | Lower | Higher |
| Fault Tolerance | Manual | Built-in |
| Use Cases | Backfills, periodic loads | Near-real-time ingestion |

### Interview Questions & Answers

**Q4. Why is micro-batch streaming often preferred for ingestion?**  

Micro-batch streaming provides lower latency, built-in checkpointing, and consistent incremental processing semantics. It also unifies batch and streaming logic, reducing code duplication.

**Q5. When is batch ingestion still appropriate?**  

Batch ingestion is suitable for historical backfills, low-frequency data sources, or systems where latency is not critical.

---

## 5. Multi-Hop Pipelines (Bronze / Silver / Gold)

Multi-hop architecture is a foundational Lakehouse design pattern.

- **Bronze** stores raw, immutable ingested data
- **Silver** contains cleaned, validated, and enriched data
- **Gold** provides aggregated, business-ready datasets

This separation improves data quality, debuggability, and reusability.

### Interview Questions & Answers

**Q6. Why not transform data directly during ingestion?**  

Separating ingestion from transformation ensures raw data is always preserved, enabling reprocessing, auditing, and schema evolution without data loss.

**Q7. How does multi-hop design help with debugging?**  

Issues can be isolated to a specific layer, making it easier to identify whether failures originate from ingestion, cleansing, or business logic.

---

## 6. Schema Drift and Schema Evolution Handling

Schema drift occurs when incoming data changes unexpectedly. Databricks and Delta Lake provide mechanisms to handle this safely.

Common strategies include:
- Schema inference with enforcement
- Controlled schema evolution
- Quarantining invalid records

### Interview Questions & Answers

**Q8. How does Auto Loader handle schema drift?**  

Auto Loader can infer schemas and detect changes over time. With schema evolution enabled, compatible changes can be automatically applied, while incompatible changes can be flagged or rejected.

**Q9. Why is uncontrolled schema evolution dangerous?**  

It can silently break downstream consumers, corrupt analytics, and introduce inconsistent data interpretations.

---

## 7. Incremental ETL Patterns Using Delta Lake

Incremental ETL processes only new or changed data instead of reprocessing entire datasets.

Common patterns include:
- Append-only ingestion
- Merge-based upserts
- Change Data Capture (CDC)

Delta Lake enables these patterns through ACID transactions, MERGE operations, and time travel.

### Interview Questions & Answers

**Q10. How does Delta Lake enable incremental processing?**  

Delta Lake maintains a transaction log that tracks data changes. Using MERGE, engineers can apply inserts, updates, and deletes atomically, ensuring consistency.

**Q11. Trade-off: Append-only vs MERGE-based pipelines?**  

Append-only pipelines are faster and simpler but increase storage and require downstream filtering. MERGE-based pipelines provide clean tables but are more expensive and complex.

---

## 8. Error Handling and Recovery Design

Production pipelines must be resilient to failures such as bad records, schema mismatches, or infrastructure issues.

Best practices include:
- Dead-letter queues for bad data
- Idempotent writes
- Checkpointing and replayability
- Separation of data and compute failures

### Interview Questions & Answers

**Q12. How do you handle bad records without stopping the pipeline?**  

By routing invalid records to a quarantine table while allowing valid records to continue processing.

**Q13. How does Delta help with recovery?**  

Delta’s ACID guarantees and time travel allow pipelines to roll back to a known-good state and reprocess data safely.

---

## 9. Delta Live Tables (DLT) – Fundamentals

Delta Live Tables is a declarative framework for building reliable data pipelines.

DLT provides:
- Automated dependency management
- Built-in data quality checks
- Simplified incremental processing
- Managed execution and monitoring

DLT abstracts much of the operational complexity of Spark pipelines.

### Interview Questions & Answers

**Q14. When should DLT be used instead of custom Spark jobs?**  

DLT is ideal when reliability, data quality enforcement, and simplified pipeline management are priorities.

**Q15. Trade-off: DLT vs custom Spark pipelines?**  

DLT reduces operational overhead but offers less low-level control compared to custom Spark jobs.

---

## 10. Workflows Orchestration for ETL

Databricks Workflows orchestrate jobs, manage dependencies, and schedule pipelines.

Key orchestration concepts include:
- Task dependencies
- Retry policies
- Parameterized jobs
- Monitoring and alerting

### Interview Questions & Answers

**Q16. Why is orchestration critical for ETL pipelines?**  

Without orchestration, pipelines become brittle, hard to monitor, and difficult to recover from failures.

**Q17. How do you design workflows for large pipelines?**  

By breaking pipelines into modular tasks, defining clear dependencies, and using retries and alerts to handle failures gracefully.

---

## 11. Production Scenarios

### Scenario 1: Late-Arriving Data

Late data arrives days after ingestion. The solution involves watermarking, reprocessing windows, and idempotent MERGE logic in Silver tables.

### Scenario 2: Schema Change in Source System

A new column appears in source data. Auto Loader detects the change, schema evolution applies it in Bronze, and downstream transformations are updated explicitly.

### Scenario 3: Pipeline Failure and Recovery

A job fails mid-run. Delta checkpointing ensures no partial writes, and the pipeline resumes from the last committed state.

---

## 12. Cost Considerations in ETL Pipelines

Cost drivers include:
- Always-on streaming clusters
- Unnecessary MERGE operations
- Excessive reprocessing
- Poor file sizing

Cost optimization strategies include:
- Micro-batch tuning
- Job clusters
- Auto Loader file notification mode
- Optimized Delta layouts

---

## 13. Best Practices and Common Pitfalls

### Best Practices
- Use Auto Loader for file ingestion
- Separate ingestion and transformation
- Adopt multi-hop architecture
- Design incremental pipelines
- Enforce schema and data quality checks
- Monitor and optimize continuously

### Common Pitfalls
- Full reloads instead of incremental processing
- Ignoring schema drift
- Tight coupling between layers
- Poor error handling strategies

---

## Key Takeaways

- Ingestion and ETL design are critical interview topics
- Auto Loader enables scalable, reliable ingestion
- Multi-hop pipelines improve quality and debuggability
- Delta Lake enables incremental and recoverable ETL
- DLT simplifies pipeline management
- Cost and reliability must be designed together
