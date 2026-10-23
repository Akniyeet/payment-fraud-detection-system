# Project Management, Governance, Milestones, and Risk Registry

**Task ID:** PFD-3  
**Component:** Component 10 – Project Management and Risks  
**Author:** Akniyet Maksatova (Lead Software Engineer / PM)  
**Status:** Completed & Ratified  

---

## 1. Project Infrastructure & Tooling Setup

To enforce end-to-end traceability and progressive engineering (Git Repository Requirements, Steps 1–9), the project relies on tightly coupled project management and version control systems.

### 1.1. YouTrack Agile Project Configuration
- **Project URL:** [Yakarta YouTrack – Payment Fraud Detection System](https://yakarta.youtrack.cloud/projects/PFD)
- **Sprint Schedule:**
  - **Sprint 001 (01 Oct – 24 Oct 2026):** Problem definition, literature review, SRS, architectural schematics, and verification planning (Pre-defense 1).
  - **Sprint 002 (25 Oct – 20 Dec 2026):** Microservices scaffolding, Redis feature store, LightGBM training baseline, and Go gateway ingestion (Pre-defense 2).
  - **Sprint 003 (10 Jan – 15 Apr 2027):** k6 load benchmarks, SHAP explainability pipeline, React analyst UI, and deployment verification (Pre-defense 3 & Final Defense).

### 1.2. Git Branching & Review Governance
- **Repository Baseline:** `main` serves as the single source of reviewed truth. Direct commits to `main` are restricted.
- **Branch-per-Task Policy (Step 6):** Every deliverable is created in an isolated branch prefixed with its YouTrack ID: `PFD-<ID>-<short-description>`.
- **Peer Review Policy (Step 7):** Merging into `main` requires at least one independent peer approval (`Approve`) via Pull Request. Self-approvals are prohibited.
- **Release Tagging (Step 8):** Each academic checkpoint is frozen via an annotated Git tag:
  - `pre-defense-1` (24 October 2026)
  - `pre-defense-2` (December 2026)
  - `final-pre-defense` (April 2027)

---

## 2. Allocation of Responsibilities (RACI Matrix)

The project is developed by a 3-person undergraduate engineering team under academic supervision.

**Academic Supervisor:** Khaimuldin Nursultan  
**Team Members:**
- **Maksatova Akniyet** – Team Lead / Frontend Developer
- **Mozgovoi Roman** – Backend Developer
- **Kadyrbek Askartau** – ML Developer

### Responsibility Assignment Matrix (RACI)
*Legend: **R** = Responsible (делает), **A** = Accountable (отвечает за результат), **C** = Consulted (консультирует), **I** = Informed (информируется)*

| Deliverable / Work Item | YouTrack Task | Akniyet (Lead/Frontend) | Roman (Backend) | Askartau (ML) | Supervisor |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Project Proposal & Scope** | PFD-1 | **A / R** | C | C | I |
| **Requirements Specification (SRS)** | PFD-9 | **A / R** | C | C | I |
| **Project Management & Risks** | PFD-3 | **A / R** | I | I | I |
| **Literature Review Matrix** | PFD-5 | C | **A / R** | C | I |
| **Existing Systems Benchmark** | PFD-6 | I | C | **A / R** | I |
| **Technology Trade-Off Analysis** | PFD-8 | C | **A / R** | C | I |
| **Architecture & Diagrams (C4)** | PFD-10 | C | **A / R** | C | I |
| **Verification & Evaluation Plan** | PFD-9 / PFD-7 | C | C | **A / R** | I |
| **Analyst Web Dashboard (React)** | PFD-11 | **A / R** | C | I | I |
| **Go Scoring Engine & Redis Store** | PFD-12 | C | **A / R** | C | I |
| **ML Inference Pipeline (FastAPI)** | PFD-13 | I | C | **A / R** | I |

---

## 3. Project Milestones & Next Steps

| Milestone | Target Date | Concrete Deliverables & Verification Focus | Target Status |
| :--- | :---: | :--- | :---: |
| **Milestone 1 (Pre-Defense 1)** | 24 Oct 2026 | Analytical baseline: Proposal (`PFD-1`), Literature Review (`PFD-5`), Systems Comparison (`PFD-6`), SRS (`PFD-9`), Architecture (`PFD-10`), Evaluation Plan (`PFD-9`), and Git infrastructure. | **Completed (`pre-defense-1`)** |
| **Milestone 2 (Pre-Defense 2)** | 20 Dec 2026 | Working microservices baseline: Go ingestion API running with Redis sliding windows, baseline heuristic rules engine (B1), LightGBM model training on Sparkov/IEEE-CIS, and initial CI unit/integration test suites. | Planned |
| **Milestone 3 (Intermediate Milestone)** | 28 Feb 2027 | System integration: Synchronous communication between Go gateway and Python inference service, fallback circuit breaker, and draft React dashboard. | Planned |
| **Milestone 4 (Final Pre-Defense & Defense)** | 20 Apr 2027 | Full empirical evaluation: k6 load tests (500 RPS sustained for 15 min, P99 < 200 ms), SHAP explainability audit, completed React dashboard with manual review triage, and final thesis defense. | Planned |

---

## 4. Technical Dependencies & Critical Path

### 4.1. Component Dependency Graph (Critical Path)
[FR-1: Go Ingestion API]
│
▼
[FR-2: Redis Sliding-Window Feature Store]
│
├──► [FR-3: Python ML Inference Service (FastAPI / LightGBM)]
│           │
▼           ▼
[FR-4: Tri-Tier Decision Routing Engine (ALLOW / MANUAL_REVIEW / DECLINE)]
│
├──► [Audit Logging & PostgreSQL Persistence]
│
▼
[FR-5 / PFD-11: React Analyst Review Dashboard]

---

### 4.2. External & Environmental Dependencies
1. **Datasets:** IEEE-CIS Fraud Detection dataset and Sparkov synthetic transaction logs (with customer account identifiers and timestamps for temporal feature aggregation).
2. **Infrastructure Tooling:** Docker Engine, Docker Compose, Go 1.22+, Python 3.10+, Redis 7, PostgreSQL 15, and Node.js 20+.
3. **Benchmarking Tools:** k6 (constant arrival rate executor for concurrency testing) and Prometheus/Grafana for server-side telemetry.

---

## 5. Comprehensive Risk Management Matrix

| Risk ID | Risk Description | Prob. | Impact | Severity | Actionable Mitigation Strategy |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **R1** | **Severe Class Imbalance (<1% fraud)** causing trivial majority-class classification | High | High | **Critical** | Exclude raw accuracy; optimize PR-AUC and Recall at fixed 1% False Positive Rate. Apply class-weighted loss and focal loss during LightGBM training. |
| **R2** | **Inference Latency SLA Breach (P99 > 200 ms)** under peak checkout traffic | Med | High | **High** | Partition latency budget (Redis lookup < 15 ms, ML inference < 30 ms). Benchmark via k6 step tests; optimize model using ONNX Runtime. |
| **R3** | **ML Microservice Failure or Timeout** halting transaction authorization | Med | High | **High** | Implement an automated circuit breaker in Go. If ML timeouts exceed 5 consecutive requests (>250 ms), fail open to deterministic rules within 30 ms (`DEGRADED_FALLBACK`). |
| **R4** | **Data Leakage from Temporal Features** during model training | High | High | **Critical** | Strictly enforce time-based chronological data splits (70% train / 15% validation / 15% test). Never shuffle financial time-series data randomly. |
| **R5** | **Redis Memory Exhaustion** from high velocity sliding-window aggregations | Med | Med | **Medium** | Enforce explicit Time-To-Live (TTL) policies on all sorted set keys (`cust:{id}:tx`) matching maximum feature aggregation windows (7 days). |
| **R6** | **Scope Creep before Pre-Defense Checkpoints** | High | Med | **High** | Maintain strict MoSCoW boundaries in SRS (`06_requirements_specification.md`). Defer biometric analysis, automated online retraining, and live bank rails to out-of-scope. |

---

## 6. Activity Traceability Protocol

To satisfy Step 6 and Step 9:
1. **Task Creation:** Each item originates in YouTrack with assignees, acceptance criteria, and priority.
2. **Implementation:** Code and documentation are committed to matching branches (`PFD-<ID>-<name>`).
3. **Verification:** Changes are submitted via Pull Request with a template referencing the task ID, what changed, and how it was verified.
4. **Closing:** Tasks are marked `Done` in YouTrack only after the Pull Request is approved by a peer reviewer and merged into `main`.
