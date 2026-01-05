# Chapter 10 – Machine Learning, Feature Store & MLOps in Databricks

**Databricks Runtime:** 13.x+  
**Target Roles:** Data Engineer | Machine Learning Engineer | MLOps Engineer | Analytics Engineer  
**Interview Focus:** MLflow, Feature Store, Training Pipelines, Hyperparameter Tuning, Model Deployment, Real-Time Inference, Monitoring & Drift  

---

## 1. Why ML, Feature Store & MLOps Matter

Machine learning is increasingly embedded in **real-time decision systems, dashboards, and predictive analytics**. In interviews, questions around ML pipelines and MLOps assess your ability to:

- Build reproducible ML workflows
- Manage model lifecycle (training → deployment → monitoring)
- Handle high-scale feature computation and storage
- Ensure governance, lineage, and performance

Databricks provides **MLflow and Feature Store** for managing experimentation, production ML workflows, and serving features in batch and real-time.

---

## 2. MLflow: Tracking, Registry & Deployment

MLflow is Databricks’ open-source **ML lifecycle platform**. Its key components:

- **Tracking:** Logs experiments, parameters, metrics, artifacts
- **Projects:** Defines reproducible pipelines with dependencies
- **Models:** Stores trained models with versioning
- **Registry:** Manages model lifecycle (Staging → Production → Archived)
- **Deployment:** Integrates with REST API, Databricks Jobs, or real-time serving

---

### Interview Questions & Answers

**Q1. How does MLflow ensure reproducibility?**  

MLflow Projects capture environment dependencies, code versioning, and parameters. Combined with experiment tracking, it allows rerunning an experiment with the same configuration and dataset.

**Q2. How do you manage multiple versions of a model in MLflow?**  

The MLflow Registry allows versioning. You can promote models from staging to production, rollback to previous versions, and maintain audit logs for compliance.

**Q3. Trade-offs: Experiment tracking vs full deployment**  

Tracking ensures reproducibility and metrics comparison but does not guarantee production-grade inference performance. Deployment requires additional testing, monitoring, and scalability considerations.

---

## 3. Feature Store Architecture & Offline/Online Flows

Databricks Feature Store centralizes **feature computation, storage, and serving**.

**Offline feature flow:**
- Features computed in batch pipelines
- Stored in Delta tables
- Reused for model training

**Online feature flow:**
- Features stored in low-latency key-value stores (like Redis or Delta Online)
- Served to real-time inference endpoints

Benefits:
- Consistency between training and serving
- Reduced duplication of feature computation
- Governance, lineage, and reuse across teams

---

### Interview Questions & Answers

**Q4. Why use a Feature Store instead of computing features on-the-fly?**  

Computing features on-the-fly increases latency, risks inconsistencies between training and serving, and duplicates computation. Feature Store ensures single-source-of-truth features.

**Q5. How do offline and online features differ?**  

Offline features are batch-oriented and used for model training; online features are real-time and support low-latency inference.

**Q6. Trade-offs: Storage cost vs feature freshness**  

Keeping features in real-time stores increases cost but provides up-to-date predictions; offline features reduce cost but may introduce stale predictions.

---

## 4. Training Pipelines & Reproducibility

Reproducible ML pipelines include:
- Deterministic feature engineering
- Versioned datasets and feature tables
- Parameterized training scripts
- Automated experiment tracking via MLflow

Databricks supports distributed training using Spark, Horovod, or ML frameworks (TensorFlow, PyTorch).

---

### Interview Questions & Answers

**Q7. How do you ensure reproducibility for distributed training?**  

- Version all datasets and features  
- Use MLflow Projects for environment isolation  
- Log hyperparameters and random seeds  
- Use deterministic data splits  

**Q8. What are common pitfalls in ML pipeline reproducibility?**  

- Non-versioned feature computation  
- Mutable datasets or streaming data without checkpoints  
- Hardcoded random seeds only in some workers  

---

## 5. Hyperparameter Tuning & Distributed Training

Databricks supports large-scale hyperparameter tuning:

- **Hyperopt, Optuna, or Grid/Random Search**
- **Parallel tuning with distributed workers**
- Logging all experiments and metrics in MLflow

Distributed training allows **training large models on GPUs or clusters** without bottlenecking a single machine.

---

### Interview Questions & Answers

**Q9. How is distributed hyperparameter tuning different from single-machine tuning?**  

Distributed tuning evaluates multiple parameter sets in parallel across worker nodes, reducing wall-clock time for experiments and enabling larger search spaces.

**Q10. Trade-offs: Grid Search vs Bayesian Optimization**  

- Grid Search: Exhaustive, predictable, expensive for high-dimensional spaces  
- Bayesian Optimization: Efficient, adaptive, but may miss global optima  

---

## 6. Real-Time Model Inference Patterns

Real-time inference requires **low-latency access** to features and models:

- Model deployed to Databricks Serving endpoint or REST API
- Online features served from Feature Store or key-value store
- Optionally, caching model predictions for frequently requested inputs

Patterns:
- **Direct API calls** for low concurrency
- **Batch micro-batching** for higher throughput

---

### Interview Questions & Answers

**Q11. How do you reduce latency for real-time inference?**  

- Use online feature stores  
- Load models into memory on serving endpoints  
- Use vectorized model inference (TensorRT, ONNX)  
- Avoid excessive joins during feature retrieval  

**Q12. Trade-offs: Real-time vs batch scoring**  

- Real-time: low latency, higher infrastructure cost  
- Batch scoring: cost-efficient, but predictions are delayed  

---

## 7. Serving ML Models with Low Latency

Databricks Model Serving supports:
- **REST endpoints** for HTTP requests
- **Autoscaling clusters** for concurrent requests
- **Versioned models** for canary deployments and rollback

Optimizations:
- Photon for vectorized scoring (if supported)  
- Feature caching  
- Load balancers for high availability  

---

### Interview Questions & Answers

**Q13. How do you implement model rollback safely?**  

Deploy new model in staging, route a small percentage of traffic, monitor metrics, and switch traffic back to previous version if needed. MLflow Registry simplifies version management.

**Q14. Trade-offs: Multiple model versions vs single production model**  

Multiple versions enable experimentation and A/B testing but increase complexity. Single model simplifies maintenance but reduces flexibility for testing.

---

## 8. Monitoring Drift, Quality, and Lineage

Key components for production ML monitoring:
- **Data drift detection** (feature distributions)
- **Model performance metrics** (accuracy, F1-score)
- **Lineage tracking** via MLflow and Feature Store
- **Alerts** for out-of-spec predictions

Monitoring ensures **reliable and trustworthy ML systems** in production.

---

### Interview Questions & Answers

**Q15. Why is monitoring feature drift important?**  

If features drift from training data, model predictions can degrade, leading to incorrect business decisions.

**Q16. How can you monitor model performance in real-time?**  

- Track metrics on incoming predictions versus ground truth (if available)  
- Use rolling evaluation windows  
- Trigger alerts when metrics fall below thresholds  

**Q17. Trade-offs: Continuous monitoring vs periodic evaluation**  

Continuous monitoring is more responsive but consumes more compute; periodic evaluation is cheaper but may detect issues late.

---

## 9. Production Scenarios

### Scenario 1: End-to-End Reproducible Training

Offline features are computed daily, logged in MLflow, and models are trained on distributed clusters. Trained models and feature lineage are stored in MLflow Registry and Feature Store.

### Scenario 2: Real-Time Fraud Detection

Incoming transactions require low-latency scoring. Online feature store provides features; models deployed via Databricks Serving respond within milliseconds. Drift monitoring alerts on anomalous feature distributions.

### Scenario 3: Multi-Team Feature Reuse

Multiple ML teams use shared feature tables. Feature Store ensures consistency, avoids recomputation, and logs feature lineage for auditability.

---

## 10. Best Practices & Common Pitfalls

### Best Practices
- Version datasets, features, and models for reproducibility  
- Use MLflow for experiment tracking and registry  
- Separate offline and online feature pipelines  
- Monitor drift and model performance continuously  
- Deploy models with autoscaling and caching for low latency  
- Leverage distributed training for large datasets and hyperparameter tuning  

### Common Pitfalls
- Hardcoding features or model parameters  
- Using inconsistent feature computation between training and serving  
- Ignoring drift detection until model fails  
- Deploying unmonitored models in production  
- Overlooking cost of real-time inference at scale  

---

## Key Takeaways

- MLflow and Feature Store are central to reproducible, production-grade ML  
- Offline and online features ensure consistency and low-latency predictions  
- Hyperparameter tuning and distributed training optimize model quality and training efficiency  
- Real-time inference requires careful feature serving, caching, and monitoring  
- MLOps practices including versioning, monitoring, and drift detection are critical for reliable ML systems  

