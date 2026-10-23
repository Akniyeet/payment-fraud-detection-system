# Real-Time Payment Fraud Detection System Using Machine Learning

[![Pre-defense Status](https://img.shields.io/badge/Milestone-Pre--defense%201%20(Oct%2024)-blue.svg)]()
[![Repository Architecture](https://img.shields.io/badge/Architecture-Go%20%7C%20FastAPI%20%7C%20Redis%20%7C%20React-green.svg)]()

A high-throughput, low-latency distributed decision platform designed to screen streaming payment transactions against financial fraud in real time[cite: 7]. The system combines deterministic heuristic rule evaluations with gradient-boosted machine learning inference (LightGBM) under strict banking latency SLAs[cite: 7].

---

## 1. Project Overview & Problem Statement

Legacy transaction monitoring engines in retail banking rely heavily on static heuristic rules[cite: 7]. These systems present two fundamental operational flaws:
- **Excessive False Positive Rates (FPR):** Rule engines frequently exceed 70% false alarm rates, adding transaction friction and causing cart abandonment[cite: 7].
- **Latency & Scalability Constraints:** Synchronous checkout gateways require verdicts in sub-200 milliseconds, which traditional systems struggle to sustain under peak velocity bursts[cite: 7].

This project delivers a hybrid, self-hosted screening pipeline providing deterministic rule filtering, sub-30 ms ML inference, atomic sliding-window state tracking via Redis, and an interactive investigation dashboard for risk analysts[cite: 7].

---

## 2. Target Stakeholders & Personas

- **Payment Service Providers (PSPs) & Card Gateways:** Ingest payment payloads via REST and enforce synchronous decision routing before financial clearance[cite: 7].
- **Financial Institutions & Risk Leads:** Minimize manual verification backlog while maintaining strict regulatory compliance and audit logs[cite: 7].
- **Fraud Risk Analysts:** Investigate flagged transactions with interpretable SHAP feature attributions and record triage overrides (`ALLOW`, `MANUAL_REVIEW`, `DECLINE`)[cite: 7].

---

## 3. Team Composition & Academic Governance

- **Academic Supervisor:** Khaimuldin Nursultan[cite: 8]  
- **Academic Institution:** Astana IT University, Group SE-2409[cite: 8]  

| Contributor | Project Role | Assigned Deliverables & Tasks |
| :--- | :--- | :--- |
| **Maksatova Akniyet** | Team Lead / Frontend Developer | Interactive analyst review dashboard (React), project management and risk governance, requirements specification, and verification planning (**PFD-1, PFD-2, PFD-3, PFD-9, PFD-11**)[cite: 8]. |
| **Mozgovoi Roman** | Backend Developer | High-throughput Go scoring engine, Redis sliding-window feature store, system architecture diagrams, and technology trade-offs (**PFD-5, PFD-8, PFD-10**)[cite: 8]. |
| **Kadyrbek Askartau** | ML Developer | Supervised fraud classification models (LightGBM/XGBoost), Python inference microservice, and existing system benchmarks (**PFD-6, PFD-7**)[cite: 8]. |

---

## 4. Current Status & Target Deliverables (Pre-defense 1)

The current milestone focuses on architectural validation, formal requirements specification, and analytical benchmarking prior to production deployment[cite: 8]:

- **Current Stage:** `pre-defense-1` (Submission deadline: October 24, 2026)[cite: 8].
- **Core Verification Targets:**
  - **Inference SLA:** P95 < 100 ms and P99 < 200 ms under sustained load of $\ge 500\text{ RPS}$ for 15 minutes[cite: 8].
  - **Detection Accuracy:** PR-AUC $\ge 0.85$ at Recall $\ge 0.80$ and FPR $\le 4.0\%$[cite: 8].
  - **Degraded Fallback:** Seamless circuit breaking to deterministic rules within 30 ms upon ML microservice timeout or unavailability[cite: 8].

---

## 5. Technology Stack

- **Scoring Backend:** Go (Golang) — ingestion API, schema validation, and rule execution[cite: 8].
- **Machine Learning Engine:** Python, FastAPI, LightGBM, ONNX Runtime[cite: 8].
- **Feature Store & Cache:** Redis (sliding-window sorted sets `cust:{id}:tx`)[cite: 8].
- **Persistence & Audit:** PostgreSQL (transaction logs and analyst review decisions)[cite: 8].
- **Analyst Web Dashboard:** React, TypeScript, TailwindCSS[cite: 8].
- **Benchmarking & Load Testing:** k6, Prometheus, Grafana, Docker Compose[cite: 8].

---

## 6. Project Documentation & YouTrack Tracking

Task lifecycle and Git traceability are managed in YouTrack:
- **Project Board:** [Yakarta YouTrack – Payment Fraud Detection (PFD)](https://yakarta.youtrack.cloud/projects/PFD)

### Deliverables Directory (`/docs/`):
- [`01_project_proposal.md`](docs/01_project_proposal.md) – Official problem statement, objectives, and scope boundaries.
- [`02_evidence_of_relevance.md`](docs/02_evidence_of_relevance.md) – Empirical survey analysis and industrial relevance evidence.
- [`03_literature_review.md`](docs/03_literature_review.md) – Critical literature review of 15 recent scientific publications and research gap analysis.
- [`04_existing_systems_comparison.md`](docs/04_existing_systems_comparison.md) – Benchmark against commercial fraud detection platforms (Radar, Sift).
- [`05_technology_comparison.md`](docs/05_technology_comparison.md) – Architectural Decision Records (ADRs) for storage, language, and model serving.
- [`06_requirements_specification.md`](docs/06_requirements_specification.md) – Functional (FR) and non-functional (NFR) system specifications.
- [`07_architecture.md`](docs/07_architecture.md) – System context, use case flows, and component decomposition[cite: 9].
- [`08_ux_prototype_and_api.md`](docs/08_ux_prototype_and_api.md) – UX mockups, interactive analyst workflows, and OpenAPI 3.1 interface contracts.
- [`09_verification_and_evaluation_plan.md`](docs/09_verification_and_evaluation_plan.md) – Empirical baseline comparison, k6 test plans, and outcome metrics.
- [`10_project_management_and_risks.md`](docs/10_project_management_and_risks.md) – Work breakdown structure, milestones, RACI matrix, and risk mitigation.
- [`11_supervisor_feedback.md`](docs/11_supervisor_feedback.md) – Academic supervisor consultation log and review protocol.

---

## 7. Local Setup & Execution Guide

Ensure [Docker](https://www.docker.com/) (v24+) and [Docker Compose](https://docs.docker.com/compose/) (v2.20+) are installed.

### 1. Clone the repository
```bash
git clone [https://github.com/Akniyeet/payment-fraud-detection-system.git](https://github.com/Akniyeet/payment-fraud-detection-system.git)
cd payment-fraud-detection-system
```

### 2. Inspect Architecture & Diagrams
Editable PlantUML sources (`.puml`) and scalable vector graphics (`.svg`) are located in the `/architecture` directory:
```bash
ls -la architecture/
```

### 3. Verification & Test Plans
System specifications and evaluation criteria for Pre-defense 1:
- Requirements & Acceptance Criteria: [`docs/06_requirements_specification.md`](docs/06_requirements_specification.md)
- Benchmarking & Testing Plan: [`docs/09_verification_and_evaluation_plan.md`](docs/09_verification_and_evaluation_plan.md)

### 4. Containerized Environment (Milestone 2 Runtime)
The multi-container scoring pipeline (Go Ingestion API, Redis 7, FastAPI LightGBM, PostgreSQL 16, Prometheus, and Grafana) will be launched via Docker Compose:
```bash
# Planned startup command for Milestone 2
docker compose up -d --build

# Ingestion gateway health check
curl -i http://localhost:8080/healthz
```
