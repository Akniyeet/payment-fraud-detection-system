# Real-Time Payment Fraud Detection System Using Machine Learning

[![Pre-defense Status](https://img.shields.io/badge/Milestone-Pre--defense%201%20(Oct%2024)-blue.svg)]()
[![Repository Architecture](https://img.shields.io/badge/Architecture-Go%20%7C%20FastAPI%20%7C%20Redis%20%7C%20React-green.svg)]()

A high-throughput, low-latency distributed decision platform designed to screen streaming payment transactions against financial fraud in real time. The system combines deterministic heuristic rule evaluations with gradient-boosted machine learning inference (LightGBM) under strict banking latency SLAs.

---

## 1. Project Overview & Problem Statement

Legacy transaction monitoring engines in retail banking rely heavily on static heuristic rules. These systems present two fundamental operational flaws:
- **Excessive False Positive Rates (FPR):** Rule engines frequently exceed 70% false alarm rates, adding transaction friction and causing cart abandonment.
- **Latency & Scalability Constraints:** Synchronous checkout gateways require verdicts in sub-200 milliseconds, which traditional systems struggle to sustain under peak velocity bursts.

This project delivers a hybrid, self-hosted screening pipeline providing deterministic rule filtering, sub-30 ms ML inference, atomic sliding-window state tracking via Redis, and an interactive investigation dashboard for risk analysts.

---

## 2. Target Stakeholders & Personas

- **Payment Service Providers (PSPs) & Card Gateways:** Ingest payment payloads via REST and enforce synchronous decision routing before financial clearance.
- **Financial Institutions & Risk Leads:** Minimize manual verification backlog while maintaining strict regulatory compliance and audit logs.
- **Fraud Risk Analysts:** Investigate flagged transactions with interpretable SHAP feature attributions and record triage overrides (`ALLOW`, `MANUAL_REVIEW`, `DECLINE`).

---

## 3. Team Composition & Academic Governance

- **Academic Supervisor:** Khaimuldin Nursultan  
- **Academic Institution:** Astana IT University, Group SE-2409  

| Contributor | Project Role | Assigned Deliverables & Tasks |
| :--- | :--- | :--- |
| **Maksatova Akniyet** | Team Lead / Frontend Developer | Interactive analyst review dashboard (React), project management and risk governance, requirements specification, and verification planning (**PFD-1, PFD-2, PFD-3, PFD-9, PFD-11**). |
| **Mozgovoi Roman** | Backend Developer | High-throughput Go scoring engine, Redis sliding-window feature store, system architecture diagrams, and technology trade-offs (**PFD-5, PFD-8, PFD-10**). |
| **Kadyrbek Askartau** | ML Developer | Supervised fraud classification models (LightGBM/XGBoost), Python inference microservice, and existing system benchmarks (**PFD-6, PFD-7**). |

---

## 4. Current Status & Target Deliverables (Pre-defense 1)

The current milestone focuses on architectural validation, formal requirements specification, and analytical benchmarking prior to production deployment:

- **Current Stage:** `pre-defense-1` (Submission deadline: October 24, 2026).
- **Core Verification Targets:**
  - **Inference SLA:** P95 < 100 ms and P99 < 200 ms under sustained load of $\ge 500\text{ RPS}$ for 15 minutes.
  - **Detection Accuracy:** PR-AUC $\ge 0.85$ at Recall $\ge 0.80$ and FPR $\le 4.0\%$.
  - **Degraded Fallback:** Seamless circuit breaking to deterministic rules within 30 ms upon ML microservice timeout or unavailability.

---

## 5. Technology Stack

- **Scoring Backend:** Go (Golang) — ingestion API, schema validation, and rule execution.
- **Machine Learning Engine:** Python, FastAPI, LightGBM, ONNX Runtime.
- **Feature Store & Cache:** Redis (sliding-window sorted sets `cust:{id}:tx`).
- **Persistence & Audit:** PostgreSQL (transaction logs and analyst review decisions).
- **Analyst Web Dashboard:** React, TypeScript, TailwindCSS.
- **Benchmarking & Load Testing:** k6, Prometheus, Grafana, Docker Compose.

---

## 6. Project Documentation & YouTrack Tracking

Task lifecycle and Git traceability are managed in YouTrack:
- **Project Board:** [Yakarta YouTrack – Payment Fraud Detection (PFD)](https://yakarta.youtrack.cloud/projects/PFD)

### Deliverables Directory (`/docs/`):
- [`01_project_proposal.md`](docs/01_project_proposal.md) – Official problem statement, objectives, and scope boundaries.
- [`02_evidence_of_relevance.md`](docs/02_evidence_of_relevance.md) – Empirical survey analysis and industrial relevance evidence.
- [`03_literature_review.md`](docs/03_literature_review.md) – Critical literature review of 15 recent scientific publications and research gap analysis.
- [`04_existing_systems_comparison.md`](docs/04_existing_systems_comparison.md) – Benchmark against commercial fraud detection platforms (Radar, Sift).
- [`06_requirements_specification.md`](docs/06_requirements_specification.md) – Functional (FR) and non-functional (NFR) system specifications.
- [`07_architecture.md`](docs/07_architecture.md) – System context, use case flows, and component decomposition.
- [`08_technology_tradeoffs.md`](docs/08_technology_tradeoffs.md) – Architectural Decision Records (ADRs) for storage, language, and model serving.
- [`09_verification_and_evaluation_plan.md`](docs/09_verification_and_evaluation_plan.md) – Empirical baseline comparison, k6 test plans, and outcome metrics.
- [`10_project_management_and_risks.md`](docs/10_project_management_and_risks.md) – Work breakdown structure, milestones, RACI matrix, and risk mitigation.

---

## 7. Local Setup & Execution Guide

Ensure [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/) are installed.

### 1. Clone the repository
```bash
git clone [https://github.com/Akniyeet/payment-fraud-detection-system.git](https://github.com/Akniyeet/payment-fraud-detection-system.git)
cd payment-fraud-detection-system
