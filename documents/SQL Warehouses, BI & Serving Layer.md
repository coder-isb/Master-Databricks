# Chapter 9 – SQL Warehouses, BI & Serving Layer in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Analytics Engineer | BI Engineer | Platform Engineer  
**Interview Focus:** SQL Warehouses Architecture, Result & Data Caching, BI Tool Integration, High Concurrency Workloads, Dashboarding, Cost Optimization  

---

## 1. Why SQL Warehouses & Serving Layers Matter

Databricks SQL Warehouses (formerly SQL Endpoints) form the **serving layer of the Lakehouse**. They provide low-latency, interactive SQL access to curated datasets for BI, dashboards, and analytics. Interviewers evaluate your ability to design **cost-efficient, high-concurrency, and performant serving layers**.  

Key requirements in production:
- Low-latency query responses
- Support for multiple simultaneous BI users
- Integration with tools like PowerBI, Tableau, Looker
- Scalability for growing data volumes
- Cost-effective cluster management

---

## 2. SQL Warehouse Architecture

Databricks SQL Warehouses leverage Spark clusters optimized for **short-lived queries and high concurrency**. Key components:

- **Compute Layer:** Auto-scaled clusters optimized for SQL queries
- **Query Execution:** Uses Databricks Photon engine for vectorized, native code execution
- **Caching Layers:** Result caching (query results) and data caching (hot Delta tables)
- **Connection Layer:** JDBC/ODBC endpoints for BI tools

SQL Warehouses can operate in **serverless mode** or **dedicated clusters**, balancing performance and cost.

---

### Interview Questions & Answers

**Q1. How does SQL Warehouse differ from a standard Databricks cluster?**  

SQL Warehouses are tuned for low-latency queries rather than heavy batch computation. They use Photon, optimized caching, and cluster auto-scaling to support many simultaneous BI queries.

**Q2. What is Photon and why is it important?**  

Photon is Databricks’ vectorized query engine written in C++. It reduces CPU cycles, optimizes query execution plans, and improves performance for interactive SQL workloads.

---

## 3. Result Caching & Data Caching

Databricks supports two caching layers:

1. **Result Cache:** Stores results of previously executed queries. Ideal for repeated dashboard queries.
2. **Data Cache:** Stores frequently accessed Delta tables in memory or SSD, improving scan performance.

Caching can **dramatically reduce latency** for dashboards and aggregated queries.

---

### Interview Questions & Answers

**Q3. When should you use result caching vs data caching?**  

- Use **result caching** for repeated query patterns or dashboard refreshes.  
- Use **data caching** for hot tables or frequently scanned datasets across multiple queries.  

**Q4. Trade-offs?**  

Result cache is lightweight but only benefits repeated queries; data cache consumes memory/compute resources but accelerates broader workloads.

---

## 4. Direct BI Tool Integration

Databricks supports **direct connections to BI tools** via JDBC, ODBC, and REST APIs. Integration considerations:

- PowerBI: Native connector for DirectQuery or import mode
- Tableau: Connect via JDBC for live querying
- Looker/Mode: Use SQL endpoints for live dashboards
- Authentication: Unity Catalog or IAM-based identity integration

---

### Interview Questions & Answers

**Q5. What is a best practice for connecting BI tools?**  

Use **SQL Warehouses with auto-scaling and Photon** for interactive dashboards. Avoid direct cluster access, as it bypasses governance and caching optimizations.

**Q6. How does authentication work for BI tools?**  

Identity is passed via JDBC/ODBC connection. Unity Catalog ensures RBAC and row/column-level security is enforced.

---

## 5. High Concurrency Workloads

High concurrency workloads require multiple users querying the same datasets simultaneously. SQL Warehouses support:

- Auto-scaling clusters
- Photon query execution
- Query queue management
- Result caching to reduce redundant computation

---

### Interview Questions & Answers

**Q7. How do you design a SQL Warehouse for 100+ concurrent dashboard users?**  

- Enable auto-scaling clusters
- Use Photon engine
- Cache frequently accessed datasets
- Partition data to avoid skew
- Monitor query latency and adjust cluster size dynamically

**Q8. Trade-offs of high concurrency?**  

High concurrency reduces per-user cost but may increase latency. Proper caching and query optimization are critical.

---

## 6. Serving Aggregated Datasets

Aggregated datasets improve BI query performance by **precomputing summaries or rollups**:

- Aggregate by date, region, or key business dimensions
- Store as Delta tables or materialized views
- Use data cache or result cache to serve dashboards efficiently

---

### Interview Questions & Answers

**Q9. Should all aggregations be precomputed?**  

Not necessarily. Only precompute high-frequency queries. Low-frequency or ad-hoc queries can run on raw tables to save storage and processing cost.

**Q10. How does materialized view vs table aggregation affect cost?**  

Materialized views are updated automatically but may incur compute cost on refresh. Aggregated tables allow more control but require manual refresh logic.

---

## 7. Dashboarding, Alerts, and Scheduled Queries

Databricks SQL allows:

- Dashboard creation with interactive filters
- Alerts based on thresholds
- Scheduled queries for ETL or reporting

These features integrate seamlessly with BI tools or native Databricks dashboards.

---

### Interview Questions & Answers

**Q11. How do you optimize dashboards for latency?**  

- Use result caching for repeated queries
- Precompute aggregated tables for heavy metrics
- Partition large datasets by filter columns
- Limit excessive join complexity

**Q12. How to handle alerting at scale?**  

Leverage scheduled SQL queries with threshold checks and integrate with messaging systems (Slack, Teams, email). Avoid running alerts on raw, unaggregated data to reduce load.

---

## 8. Designing Cost-Efficient BI Serving Layers

Key strategies:

- **Auto-scaling clusters:** Start small, scale up during query spikes
- **Photon engine:** Maximizes CPU efficiency
- **Caching layers:** Reduce repeated computation
- **Pre-aggregated datasets:** Reduce query complexity
- **Monitoring:** Query execution metrics to adjust cluster sizing and schedule refreshes

---

### Interview Questions & Answers

**Q13. Trade-off: Always-on warehouse vs serverless/on-demand?**  

- **Always-on:** Low latency, higher cost  
- **Serverless/on-demand:** Cost-efficient, may increase query startup latency  

**Q14. How do you decide caching vs pre-aggregation?**  

- Use caching for frequently accessed data with repeated queries  
- Use pre-aggregation when queries are heavy or combine large joins/aggregations

---

## 9. Production Scenarios

### Scenario 1: High-Frequency Dashboards

100+ sales dashboards are refreshed every 5 minutes. Solution: SQL Warehouse with auto-scaling, result caching, pre-aggregated tables, and Photon for low-latency queries.

### Scenario 2: Multi-Region Reporting

Data resides in multiple regions. Use Delta Sharing or UC external locations to federate access while caching hot regional datasets in each warehouse.

### Scenario 3: Cost-Constrained BI Layer

For ad-hoc analytics, enable on-demand warehouses with small clusters, auto-scaling, and selective caching of critical dashboards only.

---

## 10. Best Practices & Common Pitfalls

### Best Practices
- Use Photon engine for all BI workloads
- Enable result and data caching for repeated queries
- Precompute aggregates for dashboards and alerts
- Use UC RBAC for governance
- Monitor warehouse metrics for scaling and cost optimization

### Common Pitfalls
- Running dashboards on raw, unoptimized tables
- Ignoring caching layers for high concurrency
- Over-provisioning warehouses leading to unnecessary cost
- Bypassing governance in direct cluster connections

---

## Key Takeaways

- SQL Warehouses are the serving layer of the Lakehouse, optimized for low-latency queries  
- Photon engine and caching layers improve performance for interactive dashboards  
- Pre-aggregated datasets reduce query complexity and cost  
- BI tools integrate natively via JDBC/ODBC with UC governance enforcement  
- Cost-efficient design requires auto-scaling, caching, and selective pre-aggregation  

