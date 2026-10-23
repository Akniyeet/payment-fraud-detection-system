# Component 08: UX Prototype, User Scenarios & API Specifications

**Milestone:** Pre-defense 1 (October 24, 2026)  
**Academic Institution:** Astana IT University, Group SE-2409  
**Team Lead / Author:** Maksatova Akniyet  
**Associated YouTrack Task:** [PFD-11](https://yakarta.youtrack.cloud/projects/PFD)  
**Interactive Artifact:** [`docs/ux_prototype.html`](ux_prototype.html)

---

## 1. Overview & Interaction Scope

The Real-Time Payment Fraud Detection System serves two distinct classes of actors:
1. **External Payment Gateways & Banking Switches (Machine-to-Machine):** High-throughput, low-latency REST API consumers requiring synchronous risk evaluation (`ALLOW`, `MANUAL_REVIEW`, `DECLINE`) within sub-100 ms SLAs.
2. **Fraud Risk Analysts & Compliance Officers (Human Interface):** Domain experts investigating flagged transactions, interpreting ML explainability factors (TreeSHAP), and managing triage overrides with an audit trail.

To demonstrate interaction fidelity prior to production deployment, an interactive single-page prototype has been developed and committed directly to the repository: **[`docs/ux_prototype.html`](ux_prototype.html)**.

---

## 2. Key Interactive Scenarios

### Scenario 1: Analyst Live Feed & Case Prioritization Queue
- **Actor:** Fraud Risk Analyst.
- **Goal:** Surface high-risk and borderline transactions in real time, prioritized by ML anomaly severity.
- **Traceability:** UC3, UC4, UC7; Requirements FR-5.
- **Key Capabilities:**
  - Live throughput KPI cards (Scored today, Manual Review rate %, Median Latency).
  - Quick-filter chips (`Needs review`, `Declined`, `All transactions`).
  - Score badge classifications: `ALLOW` (< 0.35, Green), `MANUAL_REVIEW` (0.35–0.79, Amber), `DECLINE` (≥ 0.80, Red).

### Scenario 2: Deep-Dive Investigation & Explainable AI (SHAP)
- **Actor:** Fraud Risk Analyst.
- **Goal:** Understand feature attributions behind flagged transactions and record audited override decisions.
- **Traceability:** UC5, UC6; Requirements FR-5, FR-6, NFR-5.
- **Key Capabilities:**
  - **TreeSHAP Drivers:** Interactive horizontal attribution bars showing the top 3 features pushing risk up (Red) or down (Green).
  - **Redis Sliding-Window Context:** Historical velocity display (1-hour, 24-hour, and 7-day transaction frequency and cumulative volume).
  - **Audited Decision Recording:** Mandatory audit comment field generating a structured JSON log upon `APPROVE_OVERRIDE` or `CONFIRM_FRAUD`.

### Scenario 3: Gateway REST API & Shopper Verification Flow
- **Actor:** External Payment Gateway & Retail Shopper.
- **Goal:** Return deterministic scoring verdicts before payment clearance and present adaptive challenges.
- **Traceability:** UC1, UC2; Requirements FR-1, FR-4, NFR-1, NFR-3.
- **Key Capabilities:**
  - **Synchronous API Contract:** Live preview of `POST /api/v1/transactions/score` with HTTP 200/400 schemas and `X-Idempotency-Key` headers.
  - **Shopper Step-Up Experience:** Dynamic mockup showing seamless pass-through for low risk, 3D-Secure SMS/OTP challenges for borderline cases, and clear decline instructions for blocked attempts.

### Scenario 4: Degraded Resilience & Outage Simulation
- **Actor:** System Operator / DevOps Lead.
- **Goal:** Maintain continuous payment screening when the Python ML microservice is unreachable.
- **Traceability:** UC13; Requirement NFR-3.
- **Key Capabilities:**
  - Interactive toggle simulating ML container timeouts.
  - Automatic circuit breaker tripping to "Open" state.
  - Instant transition to pure deterministic rule-based evaluation (`DEGRADED_FALLBACK`) with reduced latency (12 ms).

---

## 3. Core API Interface Contract

```yaml
openapi: 3.1.0
info:
  title: Payment Fraud Detection Ingestion API
  version: 1.0.0
paths:
  /api/v1/transactions/score:
    post:
      summary: Synchronous fraud scoring evaluation
      headers:
        X-Idempotency-Key:
          schema:
            type: string
          required: true
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [transaction_id, card_token, amount, currency, timestamp]
              properties:
                transaction_id: { type: string, format: uuid }
                card_token: { type: string }
                amount: { type: number, minimum: 0.01 }
                currency: { type: string, example: "USD" }
                timestamp: { type: string, format: date-time }
                merchant_id: { type: string }
                merchant_category_code: { type: string }
                ip_address: { type: string, format: ipv4 }
                device_fingerprint: { type: string }
      responses:
        '200':
          description: Fraud evaluation verdict returned
          content:
            application/json:
              schema:
                type: object
                properties:
                  transaction_id: { type: string }
                  decision: { type: string, enum: [ALLOW, MANUAL_REVIEW, DECLINE] }
                  risk_score: { type: number, nullable: true }
                  risk_level: { type: string, enum: [LOW, MEDIUM, HIGH, DEGRADED_FALLBACK] }
                  top_factors: { type: array, items: { type: string } }
                  latency_ms: { type: integer }
        '400':
          description: Payload validation failure
```

---

## 4. Scenario Traceability Matrix

| Prototype Scenario | Primary Actor | Architecture Use Cases (`07_architecture.md`) | Requirements Specification (`06_requirements_specification.md`) | Industry Evidence Source (`02_evidence_of_relevance.md`) |
| :--- | :--- | :--- | :--- | :--- |
| **1. Live Queue & Feed** | Fraud Risk Analyst | UC3, UC4, UC7 | FR-5 (Real-time monitoring queue, status triage, severity filters) | Interview 1 (Mitigates 70%+ false alarm fatigue via risk-ranked sorting) |
| **2. Case Review & TreeSHAP** | Fraud Risk Analyst | UC5, UC6 | FR-5, FR-6, NFR-5 (Explainable AI attributions, audited decision logging) | Interview 1 (Top-3 SHAP factors accelerate case review speed by ~3x) |
| **3. Gateway Call & Shopper View** | External Gateway & Cardholder | UC1, UC2 | FR-1, FR-4, NFR-1, NFR-3 (REST ingestion, sub-200 ms SLA, idempotency cache) | Interviews 2, 3, 4 (P99 latency SLA < 200 ms, 3DS step-up challenge) |
| **4. ML Circuit Breaker Outage** | System Operator & Analyst | UC13 | NFR-3 (High availability, deterministic rule fallback, zero checkout interruption) | Interview 2 (Zero payment dropouts during backend ML container crash) |

---

## 5. How to Run & Verify the Prototype

The interactive prototype is fully self-contained as a zero-dependency, single-file HTML/JS/CSS application:

### Local Execution Instructions:
1. Clone the repository locally:
   ```bash
   git clone [https://github.com/Akniyeet/payment-fraud-detection-system.git](https://github.com/Akniyeet/payment-fraud-detection-system.git)
   cd payment-fraud-detection-system
   ```
2. Open the prototype file directly in any modern browser:
   ```bash
   # macOS
   open docs/ux_prototype.html

   # Linux
   xdg-open docs/ux_prototype.html

   # Windows
   start docs/ux_prototype.html
   ```

### Verification Checklist:
- **Tab 1 (Live Feed):** Filter transactions using `Needs review`, `Declined`, and `All transactions`. Verify that KPI counts update dynamically.
- **Tab 2 (Case Review):** Click on any pending transaction row (e.g., `tx_9f31`). Verify that top TreeSHAP factor bars (Red/Green) and Redis sliding-window metrics are displayed. Enter a reason in the audit text field and submit an `Approve (override)` or `Confirm fraud` decision.
- **Tab 3 (Gateway & Shopper):** Toggle between `Low risk`, `Medium risk`, and `High risk` chips. Observe the synchronous REST API request/response JSON payloads and the corresponding shopper view (Direct Success, 3D-Secure SMS/OTP Challenge, or Security Decline).
- **Tab 4 (ML Outage):** Click `Simulate ML service outage`. Observe the circuit breaker change from `Closed` to `Open`, the median latency drop to ~12 ms, and the scoring mode switch to deterministic rule evaluation (`DEGRADED_FALLBACK`).
- **Tab 5 (Traceability):** Confirm the formal mapping between the interactive UI, system use cases, and functional specifications.
