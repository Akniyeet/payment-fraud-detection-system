# Project Proposal and Scope: Real-Time Payment Fraud Detection System

## 1. Official Project Title
**Development of a Real-Time Payment Fraud Detection System Using Machine Learning**

---

## 2. Problem Statement
With the rapid acceleration of digital transactions and e-commerce platforms, payment fraud has evolved from basic rule-evadable tactics into coordinated, multi-vector attacks. Traditional fraud prevention relies heavily on static, hardcoded heuristic rule engines (e.g., flagging transactions over specific thresholds or originating from new IP addresses). 

These legacy mechanisms present two critical systemic failures:
1. **High False Positive Rates (FPR):** Legacy engines disproportionately flag genuine customer payments (frequently exceeding 70% of total alerts), causing unnecessary transaction friction, manual verification overhead, and direct revenue loss due to cart abandonment.
2. **High Latency and Inflexibility:** Modern e-commerce and banking gateways impose strict Service Level Agreements (SLAs), requiring transaction risk verdicts in under 200 milliseconds. Static rules fail to detect zero-day velocity fraud patterns without human intervention.

---

## 3. Target Users & Stakeholders
- **Payment Service Providers (PSPs) & Payment Gateways:** Require a low-latency scoring API to evaluate incoming transaction payloads before routing them to card networks.
- **Financial Institutions & Retail Banks:** Require automated risk-tier categorization to minimize manual audit queues.
- **Fraud Risk Analysts:** Require interpretable feature importances and transaction risk scores to investigate high-risk anomalies.
- **E-Commerce Merchants:** Benefit from lowered chargeback penalties and reduced checkout friction for legitimate shoppers.

---

## 4. Research Aim
To design, train, and benchmark an intelligent, machine-learning-driven fraud detection model capable of classifying streaming payment transactions in real time with high recall on fraudulent cases while significantly lowering false-positive rates compared to traditional heuristic benchmarks.

---

## 5. Engineering Objectives
1. **Data Preprocessing & Balancing:** Ingest benchmark financial transaction data (e.g., IEEE-CIS Fraud Detection dataset), perform feature engineering (rolling velocity, transaction amount ratios), and resolve extreme class imbalance using techniques like SMOTE and class-weighted loss functions.
2. **Model Training & Benchmarking:** Train and validate supervised learning algorithms (LightGBM, XGBoost, CatBoost) against baseline heuristic rules, prioritizing metrics like PR-AUC, ROC-AUC, and Recall at fixed low False Positive Rates.
3. **Low-Latency Inference Engine:** Package the trained pipeline into an inference engine optimized via ONNX Runtime or serialized binaries to ensure sub-200 ms end-to-end response times.
4. **High-Throughput Backend Service & API:** Implement a high-throughput, low-latency REST API service in Go, accepting transaction JSON payloads and querying the scoring service to return a decision tier (`ALLOW`, `MANUAL_REVIEW`, `DECLINE`).

---

## 6. Expected Outcomes
1. A benchmarked gradient-boosted decision model (LightGBM) achieving high Recall on fraud with a verified low False Positive Rate (<2%).
2. A containerized backend service written in Go ensuring sub-200 ms end-to-end inference latency under concurrent simulated loads.
3. An analyst-facing web dashboard (React) enabling real-time inspection of high-risk transaction flags, key feature attributions, and audit trails.
4. Comprehensive technical documentation, performance benchmarks, and reproducibility pipelines committed to the project repository.

---

## 7. Project Scope Boundaries

### In Scope
- Processing tabular transaction payloads containing cardholder velocity, transaction amount, timestamp, and device identifiers.
- Supervised binary classification modeling trained on historical transaction datasets.
- Calculation of rolling aggregation features and real-time state lookups.
- Containerized REST API service implemented in Go with scoring integration and latency benchmarking.
- Interactive Web Dashboard prototype (React) for fraud risk analysts to inspect flagged transactions and review risk metrics.
- Comprehensive architectural schematics and operational documentation.

### Out of Scope
- Direct production integration with live card processing networks (Visa, Mastercard) requiring hardware HSMs and PCI-DSS Level 1 certification.
- Complex multimodal biometric processing (e.g., fingerprint, facial recognition at checkout).
- Real-money banking settlements and dispute chargeback financial processing.
