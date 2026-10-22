# Project Proposal and Scope: Real-Time Payment Fraud Detection System

**Task ID:** PFD-1  
**Component:** Component 01 – Project Proposal and Scope  
**Author:** Maksatova Akniyet  
**Status:** Completed & Verified  

---

## 1. Administrative Overview

- **Project Title:** Development of a Real-Time Payment Fraud Detection System Using Machine Learning
- **Academic Supervisor:** Khaimuldin Nursultan
- **Academic Group:** SE-2409, Astana IT University

### Team Composition & Responsibilities

| Student Name | Role | Core Responsibilities & Assigned Tasks |
| :--- | :--- | :--- |
| **Maksatova Akniyet** | Frontend Developer | Interactive analyst web dashboard (React) for transaction inspection, project management and risk matrix, verification plan, and scope governance (**PFD-1, PFD-2, PFD-3, PFD-9, PFD-11**). |
| **Mozgovoi Roman** | Backend Developer | High-throughput Go scoring engine and REST API, Redis sliding-window feature store, system architecture diagrams, and technology trade-offs (**PFD-5, PFD-8, PFD-10**). |
| **Kadyrbek Askartau** | ML Developer | Supervised fraud classification models (LightGBM/XGBoost), class imbalance handling, Python inference microservice, and existing systems benchmark (**PFD-6, PFD-7**). |

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
- **Fraud Risk Analysts:** Require interpretable feature importances and transaction risk scores to investigate high-risk anomalies via the web dashboard.
- **E-Commerce Merchants:** Benefit from lowered chargeback penalties and reduced checkout friction for legitimate shoppers.

---

## 4. Research Aim
To design, implement, and benchmark an intelligent, machine-learning-driven fraud detection platform capable of classifying streaming payment transactions in real time with high recall on fraudulent cases while significantly lowering false-positive rates compared to traditional heuristic benchmarks.

---

## 5. Engineering Objectives
1. **Data Preprocessing & Imbalance Handling:** Ingest benchmark financial transaction data (IEEE-CIS Fraud Detection and Sparkov datasets), perform streaming feature engineering (rolling velocity, transaction amount ratios), and handle extreme class imbalance using class-weighted loss and focal loss without evaluation leakage.
2. **Model Training & Benchmarking:** Train and validate supervised learning algorithms (LightGBM, XGBoost) against baseline heuristic rules, prioritizing metrics like PR-AUC, ROC-AUC, and Recall at fixed low False Positive Rates (FPR = 1%).
3. **Low-Latency Inference Microservice:** Package the trained pipeline into an independent Python (FastAPI) inference service communicating over synchronous HTTP/JSON with a sub-30 ms model execution SLA.
4. **High-Throughput Scoring Service & API:** Implement a high-throughput, low-latency scoring service in Go, maintaining atomic rolling windows via Redis, coordinating rule execution, and returning a synchronous decision tier (`ALLOW`, `MANUAL_REVIEW`, `DECLINE`).
5. **Analyst Dashboard Interface:** Construct a responsive single-page web interface in React providing fraud analysts with queue inspection, SHAP feature visualization, and manual override capabilities.

---

## 6. Expected Outcomes
1. **Detection Quality:** A benchmarked gradient-boosted decision model (LightGBM) achieving PR-AUC ≥ 0.85 with an operational False Positive Rate < 2% (hard ceiling ≤ 4.0%).
2. **Latency & Throughput SLA:** A containerized backend service in Go ensuring P95 latency < 100 ms and P99 latency < 200 ms under a sustained load of ≥ 500 RPS for 15 minutes.
3. **High-Availability Degradation:** Graceful timeout fallback routing to deterministic rules within 30 ms if the ML microservice degrades.
4. **Analyst Inspection Interface:** A real-time analyst dashboard (React) enabling triage of `MANUAL_REVIEW` transactions, inspection of SHAP top-3 explanatory features, and audit overrides.
5. **Academic & Engineering Deliverables:** Complete documentation, reproducible Docker Compose configuration, automated testing pipelines, and verification plans committed to the project repository.

---

## 7. Project Scope Boundaries

### In Scope
- Processing tabular transaction payloads containing cardholder velocity, transaction amount, timestamp, and device identifiers.
- Supervised binary classification modeling trained on historical transaction datasets.
- Calculation of rolling aggregation features via Redis sliding windows (`cust:{id}:tx` zsets).
- Containerized REST API services implemented in Go and Python (FastAPI) orchestrated via Docker Compose.
- Interactive Web Dashboard prototype (React) for fraud risk analysts to inspect flagged transactions and review risk metrics.
- Comprehensive architectural schematics and operational verification documentation.

### Out of Scope
- Direct production integration with live card processing networks (Visa, Mastercard) requiring hardware HSMs and PCI-DSS Level 1 certification.
- Complex multimodal biometric processing (e.g., fingerprint or facial recognition at checkout).
- Real-money banking settlements and dispute chargeback financial processing.
- Dynamic automated retraining loops in production runtime.
