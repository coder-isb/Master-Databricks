
# Chapter 6 – Streaming & Real-Time Architecture in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Senior Data Engineer | Platform Engineer  
**Interview Focus:** Structured Streaming Internals, Exactly-Once Processing, State Management, Late Data Handling, Streaming Joins, Real-Time Lakehouse Design  

---

## 1. Why Streaming Architecture Is a Core Interview Topic

Real-time data processing is no longer a niche requirement. Most modern data platforms must support **near real-time ingestion, low-latency analytics, and continuous data freshness**. Interviewers use streaming questions to evaluate whether a candidate understands **distributed state management, fault tolerance, and correctness guarantees**, not just how to write streaming code.

Poor streaming design leads to:
- Data duplication or loss
- Unbounded state growth
- Incorrect aggregations
- Pipeline instability
- Excessive compute cost

A strong candidate can explain **how streaming systems recover from failures and still produce correct results**.

---

## 2. Structured Streaming – Execution Model

Structured Streaming in Databricks is built on the principle of treating streaming data as **an unbounded table**. Instead of processing individual events, Spark processes data in **micro-batches** or **continuous mode**, applying the same APIs used for batch processing.

Internally, Structured Streaming works by:
- Defining a query plan once
- Repeatedly executing it on new data
- Tracking progress using checkpoints
- Guaranteeing consistent results across failures

This unification of batch and streaming simplifies development and maintenance.

---

### Interview Questions & Answers

**Q1. How is Structured Streaming different from traditional stream processors?**

Structured Streaming provides a declarative model where developers express *what* result they want, not *how* to process each event. Spark manages scheduling, retries, and recovery using the same optimizer and execution engine as batch jobs, ensuring consistency across workloads.

**Q2. Why is the micro-batch model important?**

The micro-batch model allows Spark to reuse its batch execution engine, enabling strong fault tolerance and exactly-once semantics while still achieving near real-time latency.

---

## 3. Checkpointing & Fault Tolerance

Checkpointing is the backbone of streaming reliability. A checkpoint stores:
- Processed offsets or files
- State store data
- Progress metadata

On failure or restart, the streaming query resumes from the last successful checkpoint.

---

### Interview Questions & Answers

**Q3. What happens if a streaming job crashes mid-batch?**

Spark discards partial progress and restarts the batch from the last committed checkpoint. Since outputs are written transactionally (e.g., to Delta), no partial or duplicate data is committed.

**Q4. Why must checkpoints be stored in reliable storage?**

Checkpoint loss means loss of progress tracking, leading to reprocessing or data loss. Therefore, checkpoints must reside in durable object storage, not local disks.

---

## 4. Exactly-Once Guarantees in Databricks Streaming

Exactly-once semantics mean that each record affects the final result **once and only once**, even in the presence of failures.

Databricks achieves this by combining:
- Structured Streaming checkpoints
- Idempotent or transactional sinks
- Delta Lake ACID guarantees

---

### Interview Questions & Answers

**Q5. Is exactly-once guaranteed end-to-end?**

Exactly-once is guaranteed *between the source and the sink*, provided the sink supports transactional or idempotent writes (such as Delta Lake). External systems without these guarantees may only support at-least-once semantics.

**Q6. How does Delta Lake enable exactly-once streaming writes?**

Delta Lake writes data atomically using its transaction log. A streaming batch is either fully committed or not committed at all, preventing partial writes.

---

## 5. Stateless vs Stateful Streaming

Stateless streaming operations do not depend on past data (e.g., simple filters or projections). Stateful operations maintain intermediate state across batches (e.g., aggregations, joins, deduplication).

Stateful streaming requires careful design because state is stored and maintained over time.

---

### Interview Questions & Answers

**Q7. Why are stateful operations more expensive?**

State must be stored, updated, checkpointed, and sometimes shuffled across nodes. As state grows, memory and disk pressure increase, impacting performance and cost.

**Q8. When should stateful streaming be avoided?**

When requirements can be met using stateless transformations or batch processing, as state introduces operational complexity.

---

## 6. Late-Arriving Data, Watermarking & Deduplication

In real-world systems, events often arrive late or out of order. Structured Streaming handles this using **event-time processing** and **watermarks**.

A watermark defines how long the system should wait for late data before finalizing results.

Deduplication uses stateful tracking of event keys within a time window.

---

### Interview Questions & Answers

**Q9. What happens to data that arrives later than the watermark?**

Late data beyond the watermark is dropped from aggregations, ensuring bounded state and predictable resource usage.

**Q10. Trade-off: Large watermark vs small watermark?**

A larger watermark improves correctness but increases state size and cost. A smaller watermark reduces cost but risks dropping valid late data.

---

## 7. Auto Loader vs Structured Streaming

Although Auto Loader is built on Structured Streaming, they serve different purposes.

| Aspect | Auto Loader | Structured Streaming |
|------|------------|----------------------|
| Primary Use | File ingestion | Event stream processing |
| Source Type | Cloud object storage | Kafka, Kinesis, files |
| Scaling | Metadata/file tracking | Offset-based |
| Complexity | Lower | Higher |

---

### Interview Questions & Answers

**Q11. When should Auto Loader be used over direct streaming reads?**

Auto Loader should be used for file-based ingestion at scale, where directory listing would otherwise become a bottleneck.

**Q12. Why not use Auto Loader for Kafka streams?**

Auto Loader is designed for file discovery, not message brokers. Kafka sources already manage offsets efficiently.

---

## 8. Real-Time Delta Lake Ingestion

Delta Lake is the preferred sink for streaming pipelines in Databricks. Streaming writes to Delta tables support:
- ACID transactions
- Schema enforcement
- Exactly-once semantics
- Time travel for recovery

This enables real-time data to be queried immediately by batch and BI workloads.

---

### Interview Questions & Answers

**Q13. Why is Delta Lake ideal for streaming sinks?**

Because it guarantees atomic writes, prevents partial data, and allows downstream consumers to read consistent snapshots even while streaming ingestion is ongoing.

---

## 9. Streaming Joins & Aggregations

Streaming joins combine streaming data with:
- Static reference data
- Other streaming sources

Streaming aggregations require stateful processing and watermarking to bound state.

---

### Interview Questions & Answers

**Q14. What are the risks of streaming-stream joins?**

They require maintaining state for both streams, which can grow unbounded without proper watermarking and time constraints.

**Q15. How do you optimize streaming aggregations?**

By using event-time windows, appropriate watermarks, and minimizing state size through careful key selection.

---

## 10. Multi-Hop Streaming Lakehouse Patterns

In a streaming Lakehouse:
- **Bronze** ingests raw streaming data
- **Silver** applies cleaning, deduplication, and enrichment
- **Gold** produces aggregated, business-ready views

Each hop is typically a separate streaming query with its own checkpoint.

---

### Interview Questions & Answers

**Q16. Why separate streaming pipelines into multiple hops?**

It improves debuggability, enables reprocessing, isolates failures, and allows different SLAs per layer.

---

## 11. Production Scenarios

### Scenario 1: Duplicate Events in Kafka

Use event keys and streaming deduplication with watermarks to ensure uniqueness.

### Scenario 2: Unbounded State Growth

Introduce stricter watermarks or redesign aggregations to limit state retention.

### Scenario 3: Streaming Job Restart

Checkpointing ensures the job resumes without data loss or duplication.

---

## 12. Cost Considerations in Streaming Pipelines

Streaming cost drivers include:
- Always-on clusters
- Large state stores
- Wide watermarks
- High shuffle volumes

Cost optimization strategies:
- Right-size clusters
- Use job clusters for streaming
- Tune trigger intervals
- Limit stateful operations

---

## 13. Best Practices & Common Pitfalls

### Best Practices
- Use event-time processing
- Always configure checkpoints
- Apply watermarks to stateful operations
- Separate streaming layers
- Monitor state store metrics

### Common Pitfalls
- No watermarking
- Large unbounded state
- Mixing batch and streaming logic carelessly
- Using streaming where batch suffices

---

## Key Takeaways

- Structured Streaming unifies batch and real-time processing
- Checkpointing enables fault tolerance and recovery
- Exactly-once semantics require transactional sinks
- Stateful streaming must be carefully bounded
- Delta Lake enables real-time Lakehouse analytics
- Correctness, cost, and latency must be balanced
