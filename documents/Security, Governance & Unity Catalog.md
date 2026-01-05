
# Chapter 7 – Security, Governance & Unity Catalog in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Senior Data Engineer | Platform Engineer | Analytics Engineer  
**Interview Focus:** Unity Catalog Architecture, Centralized Governance, RBAC, Fine-Grained Access Control, Lineage, IAM Integration, Secure Data Access  

---

## 1. Why Security & Governance Are Critical Interview Topics

As organizations scale their Lakehouse platforms, **security and governance become as important as performance and correctness**. Interviewers use these topics to assess whether a candidate can design data platforms that are **secure by default, compliant, auditable, and scalable across teams**.

Weak governance leads to:
- Unauthorized data access
- Compliance violations (GDPR, HIPAA, SOC2)
- Inconsistent security rules across workspaces
- Manual, error-prone permission management
- Poor auditability and lineage gaps

A strong engineer understands that **governance must be centralized, declarative, and enforced at the platform level**, not implemented ad hoc in pipelines.

---

## 2. Unity Catalog – Architecture Overview

Unity Catalog is Databricks’ **centralized governance layer** for data and AI assets. It provides a single control plane to manage access, auditing, and lineage across **multiple workspaces and compute engines**.

Unity Catalog introduces a **three-level namespace**:

Core architectural components include:
- **Metastore** – the top-level governance boundary
- **Catalogs** – logical groupings, often aligned to business domains
- **Schemas** – databases within catalogs
- **Tables, Views, Functions, Volumes** – governed data assets

The Unity Catalog metastore is **account-level**, allowing consistent governance across all workspaces attached to it.

---

### Interview Questions & Answers

**Q1. Why was Unity Catalog introduced when Hive Metastore already existed?**

Hive Metastore is workspace-scoped and lacks fine-grained governance, lineage, and centralized auditing. Unity Catalog provides account-level governance, consistent security policies, cross-workspace access control, and built-in lineage, making it suitable for enterprise-scale Lakehouse deployments.

**Q2. What problem does the three-level namespace solve?**

It avoids name collisions, enables domain-based organization, and allows governance to be applied at multiple logical levels (catalog, schema, table) in a consistent way.

---

## 3. Centralized Governance Controls

Unity Catalog centralizes governance by enforcing policies at the **metastore level** rather than at individual clusters or notebooks.

Centralized governance enables:
- One-time policy definition
- Consistent enforcement across all compute
- Reduced operational overhead
- Strong auditability

Permissions are enforced by Databricks at query execution time, independent of user code.


### Interview Questions & Answers

**Q3. Why is centralized governance better than cluster-level controls?**

Cluster-level controls are difficult to manage at scale and easy to bypass. Centralized governance ensures policies are enforced consistently regardless of where or how data is accessed.

**Q4. How does Unity Catalog reduce operational risk?**

By eliminating duplicated permission logic in pipelines and notebooks, reducing human error, and providing a single source of truth for access policies.

---

## 4. Role-Based Access Control (RBAC)

Unity Catalog uses **ANSI SQL–based RBAC**, where permissions are granted to users, groups, or service principals.

Common privileges include:
- SELECT, INSERT, UPDATE, DELETE
- USAGE
- CREATE, MODIFY
- OWNERSHIP

RBAC is hierarchical and inherited through the object hierarchy.

---

### Interview Questions & Answers

**Q5. How is RBAC implemented in Unity Catalog?**

RBAC is implemented using SQL GRANT and REVOKE statements, applied to catalogs, schemas, tables, views, and functions. Permissions are evaluated at query time.

**Q6. Why are groups preferred over direct user grants?**

Groups simplify management, improve scalability, and reduce the risk of inconsistent permissions as users join or leave teams.

---

## 5. Row-Level and Column-Level Security

Unity Catalog supports **fine-grained access control** through:
- Row-level filters
- Column-level masking
- Dynamic views

These controls allow sensitive data to be protected without duplicating datasets.

---

### Interview Questions & Answers

**Q7. How does row-level security work in Unity Catalog?**

Row-level security is implemented using dynamic views or row filters that apply predicates based on the querying user’s identity or group membership.

**Q8. When should column masking be used instead of separate tables?**

Column masking should be used when different users need access to the same dataset but with sensitive fields obfuscated, reducing data duplication and maintenance overhead.

**Q9. Trade-off: Views vs physical data separation?**

Views reduce duplication and simplify governance but add query overhead. Physical separation improves performance but increases operational complexity.

---

## 6. Data Lineage Tracking

Unity Catalog provides **automatic, end-to-end lineage tracking** across:
- Tables and views
- Notebooks
- Jobs and pipelines
- Streaming and batch workloads

Lineage helps with impact analysis, debugging, auditing, and compliance reporting.

---

### Interview Questions & Answers

**Q10. Why is lineage important in regulated environments?**

Lineage enables organizations to demonstrate where data originated, how it was transformed, and who accessed it, which is critical for audits and compliance.

**Q11. How does lineage help during production incidents?**

It allows engineers to quickly identify downstream dependencies affected by a broken pipeline or schema change.

---

## 7. Identity Integration with Cloud IAM

Unity Catalog integrates with cloud-native identity systems:
- Azure Active Directory
- AWS IAM
- Google Cloud IAM

This enables **federated identity**, single sign-on, and consistent access control across cloud resources.

---

### Interview Questions & Answers

**Q12. Why is IAM integration important for enterprise security?**

It ensures that access to data platforms aligns with organizational identity policies, reducing credential sprawl and improving auditability.

**Q13. How do service principals fit into this model?**

Service principals represent non-human identities such as jobs and applications, enabling secure automation without shared user credentials.

---

## 8. Secure External Locations

External locations define secure access paths to cloud storage used by Unity Catalog–managed tables.

They combine:
- Storage paths
- IAM roles or credentials
- Explicit access grants

This prevents direct, uncontrolled access to raw storage.

---

### Interview Questions & Answers

**Q14. Why are external locations required in Unity Catalog?**

They enforce controlled access to storage, ensuring that only authorized principals can read or write data even if they know the underlying storage path.

**Q15. What risk does this mitigate?**

It prevents users from bypassing governance by directly accessing cloud storage outside of Databricks.

---

## 9. Secrets & Credential Passthrough

Databricks provides secure secrets management and credential passthrough to avoid hardcoding sensitive information.

- **Secrets** store tokens, passwords, and keys securely
- **Credential passthrough** allows Databricks to access data using the user’s cloud identity

---

### Interview Questions & Answers

**Q16. Why should secrets never be hardcoded in notebooks?**

Hardcoded secrets can leak through logs, version control, or notebook sharing, creating major security vulnerabilities.

**Q17. What is the benefit of credential passthrough?**

It enforces end-user identity at the data access layer, improving auditability and eliminating shared credentials.

---

## 10. Production Scenarios

### Scenario 1: Multi-Team Data Platform

Different teams require access to shared datasets with different sensitivity levels. Unity Catalog catalogs are aligned to domains, RBAC is applied via groups, and column masking protects sensitive fields.

### Scenario 2: Regulatory Audit

Auditors request proof of data access controls and lineage. Unity Catalog provides access logs, lineage graphs, and centralized policy definitions.

### Scenario 3: External Data Access

Data is stored in cloud object storage. External locations and storage credentials ensure access is controlled and auditable.

---

## 11. Cost & Operational Considerations

While Unity Catalog does not directly add compute cost, poor governance can indirectly increase cost through:
- Data duplication
- Over-permissioned access
- Inefficient security workarounds

Good governance reduces rework, audit effort, and operational overhead.

---

## 12. Best Practices & Common Pitfalls

### Best Practices
- Use Unity Catalog as the single governance layer
- Organize catalogs by business domain
- Grant permissions to groups, not users
- Use views for fine-grained access control
- Enable lineage and audit logs
- Integrate with cloud IAM

### Common Pitfalls
- Mixing Hive Metastore and Unity Catalog governance
- Hardcoding secrets
- Overusing table copies for security
- Granting excessive privileges
- Bypassing external locations

---

## Key Takeaways

- Unity Catalog enables centralized, enterprise-grade governance
- RBAC and fine-grained access control are core security primitives
- Lineage and auditing are essential for compliance
- IAM integration ensures consistent identity management
- Secure data access must be enforced at the platform level

