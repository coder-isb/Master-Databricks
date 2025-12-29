# 📘 Databricks Lakehouse Master Interview Guide – Product, GCC & Service-Based
**Version:** v5.0  
**Role:** Data Engineer / Lakehouse Architect  
**Target Companies:** Product (Microsoft, Amazon, Google), GCCs (Infosys, TCS, Capgemini), Service-Based (Accenture, Cognizant)  

**Scope:**  
- Lakehouse Architecture & Fundamentals  
- Delta Lake Concepts: ACID, Transactions, Schema, Time Travel, CDC  
- Medallion Architecture (Bronze → Silver → Gold)  
- Unified Analytics: Batch + Streaming  
- Photon Engine & Performance  
- Metadata, Unity Catalog, and Governance  
- Scenario-based, Optimization, and Interview Questions  

---

## 1️⃣ Architecture & Fundamentals

### Q1: What is Databricks Lakehouse Architecture and why was it introduced?  
Databricks Lakehouse architecture combines the **scalability and low-cost storage of a data lake** with the **management, reliability, and performance of a data warehouse**. It provides a single platform to handle structured, semi-structured, and unstructured data, supporting batch analytics, streaming pipelines, machine learning, and BI workloads. The architecture consists of three main layers: the **storage layer**, which uses Delta Lake to store data in cloud object storage with ACID guarantees and time travel; the **compute layer**, which includes Spark clusters and the Photon engine for elastic and high-performance processing; and the **services layer**, which provides metadata management, governance, and access control through Unity Catalog. By unifying these components, the Lakehouse addresses limitations of Hadoop/Spark clusters that lacked transactional consistency, streaming support, and centralized governance.  

**Follow-ups:**  
- How does the Lakehouse differ from Snowflake or traditional Hadoop clusters?  
- How would you design a multi-team analytics platform supporting batch and streaming workloads?

---

### Q2: Explain separation of storage and compute in Lakehouse.  
In Databricks Lakehouse, storage and compute are **decoupled**, enabling independent scaling. Storage resides in cloud object storage and persists all data, while compute clusters are ephemeral and can scale according to workload demand. This design provides cost efficiency because you only pay for compute when active. It also allows multiple pipelines to access the same data concurrently without resource contention. For example, nightly ETL jobs and daytime BI dashboards can run on separate clusters simultaneously, improving both performance and cost efficiency.

---

### Q3: How does Lakehouse support both batch and streaming workloads?  
Delta Lake supports **concurrent batch and streaming reads and writes**. Structured Streaming uses micro-batch processing and guarantees **exactly-once semantics**, ensuring each record is processed exactly once even during retries or failures. Snapshot isolation ensures that readers see a consistent view of the table, while multiple writers can safely update the same table concurrently. For example, IoT sensor data can be ingested in real time to the Bronze layer while historical batch data is processed to Silver and Gold layers for analytics.

---

### Q4: Differences between Lakehouse, Snowflake, and Hadoop/Spark  

| Feature | Snowflake | Hadoop/Spark | Databricks Lakehouse |
|---------|-----------|--------------|--------------------|
| Storage | Cloud-native | HDFS | Cloud Object Storage |
| Compute | Virtual Warehouses | Coupled | Elastic Spark / Photon |
| ACID | ✅ | ❌ | ✅ Delta Lake |
| Streaming | ❌ | Separate | ✅ Unified |
| Governance | Limited | Weak | ✅ Unity Catalog |

Lakehouse provides ACID guarantees, unified batch and streaming support, and centralized governance, unlike Hadoop/Spark, which often lacks transactional guarantees, or Snowflake, which is optimized for batch SQL workloads but lacks native streaming support.

---

### Q5: What is Delta Lake and why is it important?  
Delta Lake is a **transactional storage layer on top of Parquet** that provides ACID transactions, schema enforcement, time travel, and unifies batch and streaming workloads. It addresses challenges of traditional data lakes, such as inconsistent reads, lack of reliability, and no support for concurrent writes. Delta Lake maintains a **transaction log (`_delta_log`)** and ensures **snapshot isolation**, allowing multiple pipelines to read and write safely while providing auditability and versioning for compliance.

**Follow-ups:**  
- How does Delta Lake differ from plain Parquet storage?  
- How would you use Delta time travel to debug or recover a failed pipeline?

---

### Q6: Explain ACID properties in Delta Lake.  
Delta Lake implements ACID properties to ensure reliable data pipelines. **Atomicity** ensures that either all changes in a transaction are applied or none are. **Consistency** validates that table changes maintain schema integrity. **Isolation** is achieved via snapshot isolation, which allows multiple readers and writers to work concurrently without conflict. **Durability** guarantees committed data persists in storage even in case of failures. These properties make Delta Lake suitable for both batch and streaming pipelines in production environments.

---

### Q7: How does Delta Lake handle concurrent writes?  
Delta Lake uses **Optimistic Concurrency Control (OCC)**. A write first reads the latest table snapshot and applies intended changes. Before committing, Delta Lake checks for conflicts. If a conflict is detected, the commit fails and can be retried. This allows batch and streaming pipelines to safely write to the same table. For example, a streaming job updating Silver tables while a batch job aggregates metrics to Gold can coexist without corruption.

---

### Q8: Delta Transaction Log and Checkpoints  
The Delta transaction log (`_delta_log`) records all changes as JSON commit files, while checkpoints are Parquet summaries that accelerate snapshot reconstruction. Checkpoints improve query performance and allow streaming pipelines to resume from the last consistent state after failures. This mechanism ensures both reliability and efficiency for large datasets.

---

### Q9: Delta Time Travel  
Delta Lake supports querying historical versions of tables using **time travel** by version or timestamp. This feature is critical for auditing, debugging, or recovering pipelines without modifying current data. For example, if a Gold-level KPI table is accidentally overwritten, time travel can restore the table to its previous state.

---

### Q10: Schema Enforcement and Evolution  
Delta Lake enforces schema to prevent invalid writes and supports **schema evolution**, allowing new columns to be added without breaking pipelines. In streaming pipelines, the `mergeSchema=true` option lets tables adapt to incoming fields while Silver and Gold layers remain consistent for analytics.

---

### Q11: Delta Table Optimization  
Delta tables can be optimized through **partitioning**, **Z-Ordering**, and **file compaction**. Partitioning reduces the data scanned by queries, Z-Ordering clusters data within partitions for commonly filtered columns, and `OPTIMIZE` merges small files. `VACUUM` removes obsolete files, ensuring performance and reducing storage overhead. For example, Gold-level KPI tables benefit from pre-aggregation and Z-Ordering on high-cardinality columns to improve query speed.

---

### Q12: MERGE, UPSERT, Update, Delete  
Delta Lake supports `MERGE INTO` for conditional inserts, updates, and deletes, which is crucial for **CDC pipelines**. Silver tables deduplicate and validate raw Bronze data, while Gold tables aggregate it for analytics. For instance, streaming Kafka updates for customer information can be merged efficiently into Silver tables.

---

### Q13: Change Data Capture (CDC) with Delta  
CDC pipelines ingest raw changes into Bronze, deduplicate and enrich in Silver, and aggregate in Gold. Delta Lake’s **snapshot isolation and OCC** guarantee consistency across multiple streams, supporting near real-time analytics.

---

### Q14: Handling Failed Transactions  
If a transaction fails during commit, Delta Lake aborts it entirely, leaving the previous snapshot intact. Streaming jobs can resume from the last checkpoint, ensuring no data is lost or corrupted. This reliability is crucial for production-grade pipelines.

---

### Q15: Delta + Batch + Streaming Concurrency  
Delta Lake allows simultaneous batch and streaming operations. Snapshot isolation guarantees that readers see a consistent view, and OCC ensures safe writes. This enables BI dashboards to run alongside nightly ETL jobs without conflict.

---

## 2️⃣ Medallion Architecture (Bronze → Silver → Gold)

### Q16: Explain Medallion Architecture  
Medallion architecture is a layered design pattern. **Bronze** stores raw ingested data, preserving fidelity. **Silver** cleanses, deduplicates, and enriches data with reference tables. **Gold** contains curated, aggregated, and analytics-ready datasets for dashboards and ML. This layered approach improves quality, lineage, and reusability.

---

### Q17: Bronze Layer  
The Bronze layer is a raw landing zone storing minimally transformed data. It supports auditing, debugging, and late-arriving data handling. Schema evolution is applied cautiously to preserve raw records.

---

### Q18: Silver Layer  
Silver tables apply cleansing, deduplication, and enrichment. They standardize data, join reference tables, and prepare it for analytics. This layer ensures consistent and high-quality input for Gold tables.

---

### Q19: Gold Layer  
Gold tables provide aggregated, BI-ready data. Metrics and KPIs are pre-calculated to ensure fast query performance. Partitioning and caching are applied for high-performance analytics.

---

### Q20: Streaming Medallion Pipeline  
Streaming pipelines ingest raw data to Bronze, deduplicate and enrich in Silver, and maintain aggregated Gold tables. Delta Lake ensures **snapshot isolation**, **OCC**, and consistency across concurrent batch and streaming operations.

---

### Q21: Best Practices for Medallion Architecture  
- Clear separation of layers  
- Minimal transformation in Bronze  
- Validation and deduplication in Silver  
- Pre-aggregation in Gold  
- Partitioning and Z-Ordering in Silver and Gold for performance  

---

## 3️⃣ Unified Analytics: Batch + Streaming

### Q22: Exactly-Once Semantics  
Delta Lake guarantees exactly-once semantics, ensuring each record is processed only once, even under failures or retries. This is critical for dashboards, revenue reporting, and real-time analytics.

---

### Q23: Streaming Checkpoints  
Checkpoints track offsets, transaction states, and progress, allowing pipelines to resume safely after failures. They reduce latency and prevent duplicate processing.

---

### Q24: Multi-Source CDC  
Bronze ingests multiple sources, Silver deduplicates and merges using `MERGE INTO`, and Gold aggregates for analytics. Delta Lake guarantees consistency and safe concurrent operations.

---

### Q25: Monitoring & Alerting  
Monitoring involves checking query latency, throughput, and partition skew. Alerts are configured for failed batches, slow queries, and pipeline errors to maintain reliability.

---

### Q26: Handling Schema Evolution in Streaming  
Schema evolution allows adding new columns without breaking pipelines. Silver applies transformations, and Gold maintains analytics-ready tables with consistent schema.

---

## 4️⃣ Photon Engine & Performance

### Q27: What is Photon?  
Photon is a **native C++ execution engine for Spark SQL**, designed to improve query performance. It leverages vectorized execution for faster joins, aggregations, and large-scale queries.

---

### Q28: Photon vs Tungsten  
Photon outperforms Spark Tungsten for SQL-heavy workloads due to native execution. Tungsten optimizes JVM memory and CPU, while Photon provides higher throughput and lower latency for BI queries.

---

### Q29: When Photon May Not Help  
Photon is less effective for Python UDFs, ML pipelines, or custom Spark transformations, which bypass the SQL engine.

---

### Q30: Best Practices for Photon  
Use Photon for SQL-heavy queries, combine with partitioning, Z-Ordering, and caching for maximum performance.  

---

## 5️⃣ Metadata & Unity Catalog

### Q31: What is Unity Catalog?  
Unity Catalog is a centralized metadata and governance layer providing cross-workspace catalogs, fine-grained access control, row/column-level security, and lineage tracking. It enables enterprise-grade security and compliance.

---

### Q32: Unity Catalog vs Hive Metastore  
Unlike Hive Metastore, Unity Catalog provides cross-workspace sharing, fine-grained permissions, and detailed lineage tracking, making it ideal for multi-tenant Lakehouse environments.

---

### Q33: Multi-Tenant Security  
Unity Catalog allows separate catalogs, schemas, and policies per team or business unit. Row- and column-level security ensures proper isolation.

---

### Q34: Lineage Tracking  
Lineage provides visibility from source to Gold layer, supporting auditing, impact analysis, and compliance reporting.

---

## 6️⃣ Scenario-Based Questions

### Q35: Real-Time Revenue Pipeline  
Bronze ingests raw transactions, Silver deduplicates and enriches, Gold aggregates revenue metrics. Photon and Z-Ordering ensure performance.

---

### Q36: Optimize Slow Gold Queries  
Partitioning, Z-Ordering, caching hot datasets, and pre-aggregation in Silver improve Gold query performance.

---

### Q37: Failed Streaming Job Recovery  
Streaming jobs resume from last checkpoint. OCC ensures failed writes are safely retried.

---

### Q38: Delta Time Travel Use-Case  
Time travel allows recovery from failed ETL, auditing historical data, or debugging pipelines without affecting current tables.

---

### Q39: Preventing Small File Problem  
Use `OPTIMIZE` to merge small files and Z-Ordering to cluster high-cardinality columns, reducing scan times and improving performance.

---

### Q40: Multi-Region Disaster Recovery  
Replicate Delta tables across cloud regions. Checkpoints and transaction logs ensure durability and consistency for DR scenarios.

---

### Q41: Cost Optimization  
Auto-suspend idle clusters, scale compute independently, use Photon, partition and cache hot tables for efficiency.

---

### Q42: Integrating ML Pipelines  
Silver and Gold layers can serve as feature stores. Lineage and schema stability ensure high-quality inputs for ML.

---

### Q43: Monitoring & Alerting  
Track file counts, read/write throughput, partition sizes, and query latency. Alerts for failed batches and skew prevent operational issues.

---

### Q44: Handling High-Cardinality Tables  
Partition on frequently filtered columns, apply Z-Ordering on high-cardinality columns, and cache Gold tables for performance.

---

### Q45: Gold-Level KPI Design  
Aggregate metrics, partition and cache, pre-compute expensive calculations in Silver. Ensure consistency through deduplication and validation.

---

### Q46: Preventing Data Corruption  
Snapshot isolation, OCC, transaction logs, schema validation, and VACUUM prevent corruption during concurrent writes.

---

### Q47: Real-Time Analytics Dashboard Scenario  
Bronze receives event streams, Silver cleanses and deduplicates, Gold aggregates metrics. Delta Lake ensures consistent, real-time dashboards.

---

### Q48: Data Governance and Security  
Unity Catalog enforces role-based access, row/column-level security, and auditing. This ensures compliance with GDPR, HIPAA, or SOC2 requirements.

---

### Q49: Late-Arriving Data Handling  
Use Bronze to store raw events. Silver deduplicates and merges late-arriving data, Gold aggregates metrics. Delta’s ACID transactions and time travel maintain consistency.

---

### Q50: Designing Large-Scale Pipelines  
Follow layered architecture: Bronze for raw, Silver for cleansing, Gold for analytics. Optimize partitions, Z-Ordering, caching, and leverage Photon. Use Unity Catalog for governance and checkpoints for fault tolerance. Ensure batch and streaming concurrency is handled with OCC and snapshot isolation.

---


