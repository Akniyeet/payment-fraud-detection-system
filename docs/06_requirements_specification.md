# Software Requirements Specification (SRS): Real-Time Payment Fraud Detection System

**Task ID:** PFD-9  
**Component:** Component 06 – Requirements Specification  
**Author:** Akniyet Maksatova (Lead Software Engineer / System Architect)  
**System Version:** 1.0.0-alpha  
**Status:** Approved & Baseline  

---

## 1. System Overview & Scope

The **Real-Time Payment Fraud Detection System** is an enterprise-grade, high-throughput financial security microservice pipeline. Designed to bridge the gap between low-latency authorization engines and predictive machine learning models, the system ingests transaction payloads, enriches them with in-memory behavioral features, scores fraud probability, and enforces a deterministic tri-tier decision lifecycle:
- **`ALLOW`**: Automated, frictionless authorization for verified low-risk sessions.
- **`MANUAL_REVIEW`**: Step-up multi-factor challenge (3D-Secure / OTP) or triage escalation for borderline transactions.
- **`DECLINE`**: Immediate deterministic rejection for verified anomalous or malicious patterns.

---

## 2. MoSCoW Prioritization Framework

All functional and non-functional requirements are categorized according to the MoSCoW methodology:
- **Must Have (M):** Mandatory core requirements essential for system viability, latency compliance, and pre-defense delivery.
- **Should Have (S):** Critical operational capabilities scheduled for pipeline integration and analyst ergonomics.
- **Could Have (C):** High-value enhancement capabilities implemented if compute and development bandwidth permit.
- **Won't Have (W):** Explicitly excluded out-of-scope capabilities for this thesis milestone (deferred to production deployment).

| Requirement ID | Requirement Name | Type | MoSCoW Priority |
| :--- | :--- | :---: | :---: |
| **FR-1** | Real-Time Payment Transaction Ingestion API | Functional | **Must Have** |
| **FR-2** | In-Memory Sliding-Window Feature Store | Functional | **Must Have** |
| **FR-3** | Machine Learning Inference & Risk Scoring | Functional | **Must Have** |
| **FR-4** | Tri-Tier Decision Enforcement Engine | Functional | **Must Have** |
| **FR-5** | Security Analyst Review Queue & Triage Dashboard | Functional | **Should Have** |
| **FR-6** | Automated Incident Alerting & Audit Logging | Functional | **Should Have** |
| **FR-7** | Model Drift Detection & Shadow Scoring Pipeline | Functional | **Could Have** |
| **FR-8** | Autonomous Dynamic Threshold Recalibration | Functional | **Won't Have** |
| **NFR-1** | Strict Sub-200ms Inference Latency SLA (P99) | Non-Functional | **Must Have** |
| **NFR-2** | Sustained Throughput & Concurrency ($\ge 500$ RPS) | Non-Functional | **Must Have** |
| **NFR-3** | Idempotency & Fault-Tolerant Circuit Breaking | Non-Functional | **Must Have** |
| **NFR-4** | End-to-End Data Security & Payload Hashing (PCI DSS) | Non-Functional | **Must Have** |
| **NFR-5** | Machine Learning Explainability & Feature Attribution | Non-Functional | **Should Have** |

---

## 3. Functional Requirements (FR)

### FR-1: Real-Time Payment Transaction Ingestion API (Must Have)
- **Description:** The system shall expose a stateless RESTful HTTP endpoint (`POST /api/v1/transactions/score`) accepting structured JSON payloads representing card-not-present payment requests.
- **Input Schema Fields:** `transaction_id` (UUIDv4), `card_token` (SHA-256 hashed), `amount` (float64), `currency` (ISO-4217), `timestamp` (RFC-3339 UTC), `merchant_id` (string), `merchant_category_code` (MCC 4-digit), `ip_address` (IPv4/IPv6), `device_fingerprint` (hash string).
- **Payload Validation:** The API Gateway shall perform structural schema validation; malformed payloads or missing mandatory fields shall be rejected immediately with HTTP 400 Bad Request.

### FR-2: In-Memory Sliding-Window Feature Store (Must Have)
- **Description:** The system shall utilize an in-memory Redis cluster to calculate and aggregate stateful behavioral velocity features in real time without querying persistent disk-backed storage.
- **Feature Aggregations:**
  * Transaction count for `card_token` over the trailing 1-hour, 24-hour, and 7-day sliding windows.
  * Cumulative spend amount for `card_token` over the trailing 24 hours.
  * Geolocation IP-velocity (physical distance between consecutive transactions divided by elapsed time).
- **TTL Policy:** All transactional lookup keys stored in Redis shall implement an explicit Time-To-Live (TTL) expiration matching feature calculation horizons.

### FR-3: Machine Learning Inference & Risk Scoring (Must Have)
- **Description:** The core scoring engine shall execute optimized Gradient Boosted Decision Tree (LightGBM) inference to produce a continuous risk probability score $S \in [0.0000, 1.0000]$.
- **Optimization Target:** The model must prioritize Precision-Recall Area Under Curve (PR-AUC $\ge 0.85$) over raw accuracy to prevent false declines on severely imbalanced financial data (fraud rate $\le 2.5\%$).

### FR-4: Tri-Tier Decision Enforcement Engine (Must Have)
- **Description:** The system shall evaluate the computed risk score against deterministic operational thresholds to return an immutable routing decision:
  * **`ALLOW`** ($\text{Score} < 0.35$): Transaction approved for zero-friction authorization.
  * **`MANUAL_REVIEW`** ($0.35 \le \text{Score} < 0.80$): Transaction routed to step-up authentication or analyst triage queue.
  * **`DECLINE`** ($\text{Score} \ge 0.80$): Immediate rejection with standardized error response.

### FR-5: Security Analyst Review Queue & Triage Dashboard (Should Have)
- **Description:** Transactions designated as `MANUAL_REVIEW` shall be dispatched to an asynchronous PostgreSQL queue accessible through a web-based React operations portal.
- **Operator Actions:** Certified analysts can inspect transaction metadata, review top risk indicators, and manually apply an override decision (`APPROVE_OVERRIDE` or `CONFIRM_FRAUD`).

### FR-6: Automated Incident Alerting & Audit Logging (Should Have)
- **Description:** All evaluated transactions and decisions must be logged to an append-only, tamper-evident audit log with structured JSON output. Rejections exceeding $\$2,500$ USD shall emit asynchronous alerts to security channels.

### FR-7: Model Drift Detection & Shadow Scoring Pipeline (Could Have)
- **Description:** The ingestion pipeline shall support dual scoring, evaluating live transactions against both the active baseline model and an experimental candidate model to monitor performance degradation.

---

## 4. Non-Functional Requirements (NFR)

### NFR-1: Real-Time Latency SLA (Must Have)
- **Specification:** The total end-to-end response time of the scoring service (measured from HTTP request receipt to HTTP response emission) shall not exceed **200 milliseconds at the 99th percentile (P99)** under standard operational load.
- **Component Budget:**
  * Ingestion & schema validation: $\le 10\text{ ms}$
  * Redis velocity feature lookup: $\le 15\text{ ms}$
  * LightGBM ML inference: $\le 45\text{ ms}$
  * Gateway routing & response serialization: $\le 15\text{ ms}$
  * Residual network buffer: $\le 115\text{ ms}$

### NFR-2: Throughput & Concurrent Scalability (Must Have)
- **Specification:** The Golang HTTP API gateway shall sustain a minimum throughput of **500 Requests Per Second (RPS)** with zero message drop or memory leak during sustained 15-minute load testing.

### NFR-3: Idempotency & Fault-Tolerant Circuit Breaking (Must Have)
- **Idempotency:** The API must mandate an `X-Idempotency-Key` header. Repeated requests bearing identical keys within 24 hours must return cached verdicts without re-triggering inference or duplicate metric increments.
- **Circuit Breaker:** If the ML inference engine experiences consecutive execution timeouts (>250 ms) across 5 sequential requests, the gateway must trip an automated circuit breaker into **Fail-Open Mode** (returning `ALLOW` with `risk_level: DEGRADED_FALLBACK`) to avoid halting merchant checkout flows.

### NFR-4: Data Security, Tokenization & Regulatory Compliance (Must Have)
- **PCI DSS Compliance:** Primary Account Numbers (PAN) and CVV codes shall **never** be accepted, stored, or processed in plain text. Only tokenized strings or irreversibly salted cryptographic hashes (`SHA-256`) are permitted.
- **Transport Security:** All external and internal service-to-service communications must enforce TLS 1.3 encryption.

### NFR-5: Machine Learning Explainability & Feature Attribution (Should Have)
- **Specification:** For every transaction classified as `MANUAL_REVIEW` or `DECLINE`, the system must generate local feature importance indicators (e.g., TreeSHAP values) identifying the top-3 features driving the risk classification.

---

## 5. Verifiable Acceptance Criteria Matrix (Verification Protocol)

To satisfy academic thesis and pre-defense verification standards, every mandatory (**Must Have**) requirement is mapped to a concrete, reproducible verification method and pass/fail metric threshold:

| Req ID | Target Metric / Threshold | Verification Method | Verifiable Acceptance Test Case (Given-When-Then) | Pass/Fail Criteria |
| :--- | :--- | :--- | :--- | :--- |
| **FR-1** | HTTP 200 on valid JSON; HTTP 400 on malformed schema | Automated Integration Test (`Go test`) | **Given** a payment payload missing the mandatory `card_token` field, **When** posted to `/api/v1/transactions/score`, **Then** the service returns HTTP 400 Bad Request within 10 ms. | Passes if schema rejects 100% of invalid payloads without engine crashes. |
| **FR-2** | Feature retrieval time $\le 15\text{ ms}$; TTL expiration verified | Benchmark Test (`Go bench`) & Redis CLI | **Given** 10 simulated card transactions within 60 seconds, **When** sliding window queries run, **Then** Redis increments `card_1hr_count` to 10 and expires after 3600 seconds. | Passes if Redis velocity values match exact transaction count and respond in $<15\text{ ms}$. |
| **FR-3** | Model PR-AUC $\ge 0.85$; False Positive Rate $\le 4.0\%$ | Offline Validation Script (`Python / Scikit-learn`) | **Given** the IEEE-CIS Fraud test dataset split ($N=100,000$), **When** evaluated against the LightGBM classifier, **Then** PR-AUC exceeds 0.85 at recall $\ge 0.80$. | Passes if calculated PR-AUC metric strictly meets or exceeds 0.850. |
| **FR-4** | Deterministic 3-way routing (`ALLOW` / `MANUAL_REVIEW` / `DECLINE`) | Automated Unit Tests (`Go test`) | **Given** mocked model probabilities of `0.20`, `0.55`, and `0.85`, **When** processed by the rules engine, **Then** responses return `ALLOW`, `MANUAL_REVIEW`, and `DECLINE` respectively. | Passes if 1,000 synthetic test cases achieve 100% boundary mapping accuracy. |
| **NFR-1** | End-to-end P99 response time $< 200\text{ ms}$ under load | Automated Load Test (`k6` / `Vegeta`) | **Given** a sustained load of 500 concurrent virtual users generating transactions over 10 minutes, **When** latency percentiles are aggregated, **Then** P99 response latency is $< 200\text{ ms}$. | Passes if `k6` summary reports `http_req_duration{p(99)} < 200ms`. |
| **NFR-2** | Sustained Throughput $\ge 500\text{ RPS}$; 0% drop rate | Distributed Stress Test (`k6`) | **Given** a constant arrival rate of 500 RPS over 15 minutes, **When** requests execute against the scoring container, **Then** the service successfully completes all requests with 0% error rate. | Passes if total processed HTTP 200/400 responses $\ge 450,000$ with zero 500/502 errors. |
| **NFR-3** | 100% idempotency retention; Circuit breaker trips $\le 5$ errors | Automated Fault Injection Test | **Given** a payload sent with key `IDEMP-001`, **When** the identical request is re-sent 5 seconds later, **Then** the service returns HTTP 200 with the identical score without re-invoking ML inference. | Passes if secondary request resolves in $<5\text{ ms}$ directly from Redis cache. |
| **NFR-4** | Zero unhashed PAN/CVV leakage in logs or database | Static Code Analysis & Regex Log Audit | **Given** 10,000 test transactions executed through the pipeline, **When** log dumps and Redis memory dumps are scanned with regex `\b(?:\d{4}-?){3}\d{4}\b`, **Then** zero matches are found. | Passes if regex scanner detects exactly 0 occurrences of plain-text card data. |

---

## 6. Requirements Change Management & Traceability

1. **Upstream Traceability:** Every functional and non-functional requirement originates directly from empirical survey findings and expert interview protocols documented in `docs/02_evidence_of_relevance.md`.
2. **Downstream Traceability:** All future pull requests containing code for the Golang backend (`/services/gateway`) and ML inference service (`/services/inference`) must reference these explicit Requirement IDs (e.g., `FR-1`, `NFR-1`) in their commit messages and PR acceptance checklists.
