# Chapter 14 – External Integrations & Ecosystem Architecture in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Integration Engineer | Platform Engineer | Analytics Engineer  
**Interview Focus:** Event streaming, ETL/ELT tools integration, API interfaces, Delta Sharing, federation, ML system connectivity, and downstream system exports  

---

## 1. Why External Integrations Matter

Databricks rarely exists in isolation in modern Lakehouse architectures. **Integration with streaming platforms, ETL tools, ML systems, and external query engines** is critical to enable:

- Real-time analytics  
- Multi-source data ingestion  
- Cross-organizational data sharing  
- Seamless ML model deployment and feature consumption  
- Federation of queries with other cloud data systems  

Interviewers expect you to **understand both technical integration mechanisms and trade-offs** between cost, latency, and reliability.

---

## 2. Kafka, Event Hubs, Kinesis Integration

Databricks supports streaming ingestion from **Kafka, Azure Event Hubs, and AWS Kinesis** via **Structured Streaming**:

- Supports **exactly-once delivery**, checkpoints, and watermarking  
- Ideal for real-time dashboards, monitoring, and ML pipelines  
- Integration uses **connectors** or **built-in streaming APIs**  

---

### Interview Questions & Answers

**Q1. How do you ingest real-time events from Kafka into Delta Lake?**  

- Configure a Spark Structured Streaming source with Kafka bootstrap servers  
- Define topic(s) and offsets  
- Apply schema and transformations  
- Write to Delta Lake with checkpointing for fault tolerance  

**Q2. Trade-offs: Kafka vs Kinesis vs Event Hubs**  

- Kafka: High throughput, open-source, more management overhead  
- Kinesis: AWS managed, lower operational overhead, cost-effective at smaller scale  
- Event Hubs: Azure managed, integrated with cloud ecosystem, limited throughput per unit  

---

## 3. Fivetran, Informatica, Matillion Ingestion

These ETL/ELT tools provide **low-code ingestion** for batch or near-real-time data:

- Connect to source systems (databases, APIs, SaaS apps)  
- Transform and stage data in Bronze/Silver/Gold layers in Delta  
- Handle schema drift, incremental loads, and CDC (Change Data Capture)  

---

### Interview Questions & Answers

**Q3. When would you use Fivetran vs Databricks Autoloader?**  

- Fivetran: SaaS connectors, easy maintenance, less coding  
- Autoloader: Custom streaming ingestion, cloud file-based ingestion, Delta Lake optimized  

**Q4. Trade-offs: Managed ETL vs self-managed pipelines**  

Managed ETL reduces engineering effort but limits customization and may be costlier. Self-managed pipelines offer flexibility but require more operational overhead.

---

## 4. APIs & External System Interfaces

Databricks can **ingest from or export to APIs**:

- Use Python/Scala/REST APIs for batch or streaming ingestion  
- Integrate with internal or external web services  
- Apply transformations and load results to Delta Lake or downstream systems  

---

### Interview Questions & Answers

**Q5. How do you design an API-based ingestion pipeline?**  

- Poll or subscribe to external API endpoints  
- Transform response into Delta Lake-compatible format  
- Use checkpointing to ensure exactly-once processing  

**Q6. Trade-offs: API polling vs event-based ingestion**  

Polling is simpler but may cause latency or excessive API calls. Event-driven ingestion is real-time but requires robust event-handling architecture.

---

## 5. Delta Sharing Across Organizations

**Delta Sharing** enables **secure, cross-organization sharing**:

- Share live Delta tables without copying data  
- Readable by any Delta Sharing-compliant client  
- Ensures governance and fine-grained access control  

---

### Interview Questions & Answers

**Q7. How does Delta Sharing work?**  

- Publisher defines share with tables and access rules  
- Consumer accesses shared data via token-based authentication  
- No data duplication; updates are live  

**Q8. Trade-offs: Delta Sharing vs traditional ETL exports**  

Delta Sharing provides near real-time sharing, reduces duplication, and enforces governance. ETL exports copy data, may lag, and increase storage cost.

---

## 6. Federation with External Query Engines

Databricks supports **federated queries**:

- Query external databases (Snowflake, Redshift, SQL Server) from within Databricks  
- Use **Spark JDBC connectors** or **Databricks SQL federation**  
- Useful for hybrid Lakehouse/data warehouse scenarios  

---

### Interview Questions & Answers

**Q9. How do you federate queries across external engines?**  

- Configure JDBC/ODBC connections  
- Use Delta tables as the canonical source and join with external datasets  
- Optimize for pushdown predicates to reduce data movement  

**Q10. Trade-offs: Federated query vs data ingestion**  

Federated queries avoid duplication but may be slower due to network latency and lack of native caching. Ingested data is faster but consumes storage.

---

## 7. Integration with ML Systems (SageMaker, Azure ML)

Databricks integrates with ML systems for **model training, scoring, and feature consumption**:

- Use MLflow to track experiments and deploy models  
- Access Delta Lake features for real-time or batch inference  
- Enable **feature store consumption** in SageMaker or Azure ML  

---

### Interview Questions & Answers

**Q11. How do you integrate Databricks features with SageMaker?**  

- Extract features from Delta Feature Store  
- Train models in SageMaker using exported datasets  
- Optionally deploy models in Databricks for low-latency inference  

**Q12. Trade-offs: Model training in Databricks vs external ML platform**  

Databricks offers simplicity and native Delta integration. External platforms provide specialized frameworks and distributed training capabilities but require data movement.

---

## 8. Downstream System Exports

Databricks can export data to:

- **Data warehouses:** Snowflake, Redshift  
- **BI tools:** Tableau, Power BI, Looker  
- **Cloud object storage:** S3, ADLS, GCS  
- **Other systems:** APIs, ML platforms  

**Optimizations:**

- Use **Delta Live Tables** for transformation before export  
- Aggregate and cache large datasets to reduce export cost  
- Incremental exports to avoid full-table movement  

---

### Interview Questions & Answers

**Q13. How do you optimize data exports to downstream systems?**  

- Export only incremental changes  
- Pre-aggregate large datasets  
- Use Delta caches or Photon for fast query performance  
- Monitor throughput and API limits  

**Q14. Trade-offs: Full export vs incremental export**  

Full export is simpler but costly and time-consuming. Incremental export reduces cost and time but requires tracking change data and state.

---

## 9. Production Scenarios

### Scenario 1: Real-Time Analytics

Streaming events from Kafka and Event Hubs are ingested into Bronze tables, transformed in Silver, and published to a shared Delta Lake for multiple downstream analytics platforms.

### Scenario 2: Cross-Org Data Sharing

Company shares curated datasets with partners via Delta Sharing. Partners query live data without duplicating storage. Governance ensures row-level access control.

### Scenario 3: Hybrid ML Pipeline

Feature store in Databricks feeds SageMaker for model training. Real-time scoring pipelines in Databricks consume the same features for low-latency predictions.

---

## 10. Best Practices & Common Pitfalls

### Best Practices
- Use native connectors for Kafka, Event Hubs, and Kinesis for exactly-once processing  
- Prefer Delta Sharing over manual data exports for cross-organization sharing  
- Use federated queries for hybrid architectures to avoid unnecessary duplication  
- Monitor API limits and network latency when integrating external systems  
- Incremental exports reduce cost and improve pipeline performance  

### Common Pitfalls
- Hardcoding API keys or credentials in notebooks  
- Ignoring schema drift in external sources  
- Overloading federated queries without pushdown optimization  
- Exporting full datasets unnecessarily, increasing storage and network costs  
- Neglecting governance when sharing data externally  

---

## Key Takeaways

- Databricks enables robust integration with streaming platforms, ETL tools, APIs, and ML systems  
- Delta Sharing allows secure cross-organization collaboration without data duplication  
- Federated queries enable hybrid architectures while minimizing storage overhead  
- Incremental pipelines and pre-aggregations reduce export costs and latency  
- Following best practices ensures reliability, scalability, and secure external data sharing  

