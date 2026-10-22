# Verification and Evaluation Plan – Real-Time Payment Fraud Detection System

**Task ID:** PFD-9  
**Component:** Component 09 – Verification and Evaluation Plan  
**Author:** Askartau Kadyrbek  
**Status:** Under Review  

---

## 1. Purpose

The project aim is achieved if the ML-driven system catches more fraud than a heuristic rules baseline at the same false-positive rate, and still returns each decision within the latency SLA. This plan defines how that is demonstrated: the baselines, outcome metrics, measurement conditions, pass criteria, and the tests that verify each requirement.

Requirement IDs (`FR-*`, `NFR-*`) and their acceptance thresholds come from the requirements specification [`06_requirements_specification.md`](06_requirements_specification.md) (SRS). Objectives and expected outcomes come from [`01_project_proposal.md`](01_project_proposal.md). Decision values are `ALLOW` / `MANUAL_REVIEW` / `DECLINE` (FR-4).

## 2. What must be demonstrated

Hypotheses H1–H3 are taken from the team project plan. They are tested with numbers, not promised in advance.

| # | Claim | Source | Pass criterion |
|---|-------|--------|----------------|
| E1 | ML-driven scoring catches more fraud than the rules baseline at the same false-positive rate | H1, proposal objective 2 | Recall(ML) > Recall(rules) at FPR = 1 %, and the 95 % bootstrap CI of the difference excludes 0 |
| E2 | Combining rules with the model is not worse than the model alone | H2 | PR-AUC(rules + ML) ≥ PR-AUC(ML only) − 0.01; higher recall on injected attack scenarios |
| E3 | The model meets the SRS quality threshold | FR-3 | PR-AUC ≥ 0.85 at recall ≥ 0.80 and FPR ≤ 4.0 % on the IEEE-CIS test split |
| E4 | False positives stay low at the operating thresholds | Proposal expected outcome 1 | FPR < 2 % (target); FPR ≤ 4.0 % (SRS hard limit, FR-3) |
| E5 | Decisions are fast enough for synchronous checkout | H3, NFR-1, NFR-2 | P99 < 200 ms (NFR-1) and P95 < 100 ms (internal target) at a constant ≥ 500 RPS for 15 min with 0 % errors (NFR-2) |
| E6 | Checkout does not stop when the ML engine fails | NFR-3 | During an ML outage under load: 0 HTTP 5xx, circuit breaker trips after ≤ 5 consecutive timeouts, responses marked `DEGRADED_FALLBACK` |
| E7 | The conclusion does not depend on one dataset | Proposal objective 1 | E1 holds in the same direction on both Sparkov and IEEE-CIS |

If a claim is not confirmed, this is reported as a result, with the measured numbers and an analysis of why.

## 3. Baselines for comparison

All configurations are run **by the same service** on the same test data. Only the decision logic changes, so the comparison is fair.

| ID | Configuration | What it represents | How it is configured |
|----|---------------|--------------------|----------------------|
| B0 | "Allow all" | Lower bound; sanity check for the metrics | Every transaction returns `ALLOW`; recall = 0 |
| B1 | Rules only | Traditional heuristic engine from the problem statement (main baseline) | Static rules (night + large amount, velocity per hour, amount spike vs. customer average, impossible travel, new merchant + large amount, burst of small payments); thresholds tuned on the **training** split only |
| B2 | Logistic regression | Simple ML baseline | Same feature vector as M1, class weights, thresholds tuned on validation |
| M1 | LightGBM (ML only) | Proposed model | Optuna, 50–100 trials on validation; class-weighted loss for imbalance |
| M2 | Rules + LightGBM | Proposed decision engine (FR-4) | Hard rules for critical patterns, model score for the rest; combination chosen on validation |

## 4. Data and splits

| Item | Decision |
|------|----------|
| Main dataset | Sparkov Credit Card Transactions (~1.85 M tx, ~0.5 % fraud): has customer ID, time, merchant and location, so the sliding-window features of FR-2 can be computed |
| Second dataset | IEEE-CIS Fraud Detection (~590 K tx, ~3.5 % fraud): real anonymised data; used for the FR-3 acceptance test and to repeat E1 |
| Split | By time, not random: first ~70 % train, next ~15 % validation, last ~15 % test |
| Test split | Not used until the final experiments (April 2027). All thresholds, rule parameters and hyper-parameters are chosen on train/validation |
| Labels | The generator sends transactions without the fraud label and stores labels separately; the scoring service never sees them |
| Warm-up | Before each test run, Redis and PostgreSQL are cleared and customer history is rebuilt from train + validation, so customers have history at the start of the test |

## 5. Outcome metrics

Notation: TP = fraud caught (`MANUAL_REVIEW` or `DECLINE`), FP = legitimate payment flagged, FN = fraud missed (`ALLOW`), TN = legitimate payment allowed.

```latex
\text{Recall} = \frac{TP}{TP + FN} \qquad
\text{Precision} = \frac{TP}{TP + FP} \qquad
\text{FPR} = \frac{FP}{FP + TN} \qquad
\text{Prevented loss} = \frac{\sum \text{amount of caught fraud}}{\sum \text{amount of all fraud}}
```

### 5.1 Detection quality

| Metric | Why it is used | Primary? |
|--------|----------------|:--------:|
| PR-AUC | Threshold-independent quality under strong class imbalance; SRS optimisation target (FR-3) | ✅ |
| Recall at FPR = 1 % | Main comparison point: "how much fraud is caught when 1 in 100 honest customers is flagged" | ✅ |
| Recall, Precision, FPR at the operating thresholds | What the deployed system actually does | |
| Prevented loss, % | Translates results into money | |
| `MANUAL_REVIEW` rate, % | Analyst workload; must not grow unbounded | |
| ROC-AUC | Reported for comparison with the literature only | |

Accuracy is **not** used: with < 1 % fraud, "allow all" (B0) already scores > 99 %.

### 5.2 Performance and robustness

| Metric | Source | Requirement |
|--------|--------|-------------|
| End-to-end latency P50 / P95 / P99, ms | k6 (client side) and Prometheus histogram (server side) | NFR-1 |
| Sustained throughput, RPS, and RPS at which P99 crosses 200 ms | k6 step test | NFR-2 |
| Error rate (HTTP 5xx, timeouts), % | k6 | NFR-2 |
| Feature lookup time in Redis, ms | Go benchmark | FR-2 |
| ML timeout count, circuit-breaker trips, recovery time, s | Prometheus counters, fault test | NFR-3 |

### 5.3 Explainability

| Metric | Method | Requirement |
|--------|--------|-------------|
| Share of flagged transactions with top-3 SHAP features | Automated check on all `MANUAL_REVIEW` / `DECLINE` decisions of the test run | NFR-5 |
| Faithfulness of explanations | 50 random flagged transactions checked by hand: listed reasons match the actual top SHAP features and fired rules | NFR-5 |

## 6. Measurement conditions

| Condition | Value |
|-----------|-------|
| Hardware | One laptop; CPU model, cores, RAM and OS recorded in the results file |
| Deployment | All services in Docker containers on that laptop, no other heavy processes running |
| Load tool | k6, constant-arrival-rate executor |
| Load profiles | 100 / 500 / 1,000 / 2,000 RPS; 1 min warm-up (not measured) + 5 min measured per level; plus one 15-min run at 500 RPS for NFR-2 |
| Repetitions | 3 runs per profile; the median run is reported, min–max shown as error bars |
| Quality runs | Generator replays the full test split once per configuration (B1, B2, M1, M2) at maximum speed |
| Statistical confidence | 95 % CI by bootstrap (1,000 resamples of the test split); differences between configurations by paired bootstrap on the same transactions |
| Version tracking | Every run records the git commit, rule-file hash and model version in the results file |

Quality metrics are computed by one SQL query that joins stored decisions with the separately stored labels, so every number in the thesis can be reproduced from the database.

## 7. Testing plan

| Level | What is checked | Tool | Example / pass criterion | Verifies | From milestone |
|-------|-----------------|------|--------------------------|----------|----------------|
| Unit | Decision routing, rules, feature formulas | `go test` (table-driven), `pytest` | Model scores 0.20 / 0.55 / 0.85 → `ALLOW` / `MANUAL_REVIEW` / `DECLINE`; 1,000 synthetic cases, 100 % correct boundary mapping | FR-4 | M2 |
| API validation | Schema checks on the scoring endpoint | Go integration test | Payload without `card_token` → HTTP 400 within 10 ms; 100 % of invalid payloads rejected without crashes | FR-1 | M2 |
| Integration | Feature store on a real Redis | testcontainers-go, `go test -bench` | 10 tx in 60 s → `card_1hr_count` = 10, key expires after 3,600 s, lookup < 15 ms | FR-2 | M2 |
| Idempotency | Same idempotency key sent twice | Integration test | Second request returns the identical score in < 5 ms from Redis, ML not called again | NFR-3 | M2 |
| Contract | Go and Python agree on the request/response format | OpenAPI + tests on both sides | Renamed field → test fails | FR-1, FR-3 | M2 |
| Feature parity | Serving features = training features | Comparison script on 1,000 tx | Difference ≤ 0.01 % | FR-2, FR-3 | M2 |
| Offline model validation | Model quality on held-out data | Python validation script | PR-AUC ≥ 0.85 at recall ≥ 0.80, FPR ≤ 4.0 % (IEEE-CIS test) | FR-3 | M3 |
| Security | No card data in logs or storage | Regex scan of logs and Redis/DB dumps after 10,000 tx | 0 matches of plain-text PAN/CVV | NFR-4 | M2 |
| Audit logging | Every decision is logged | Integration test | Each scored transaction has one audit record with decision and score | FR-6 | M3 |
| End-to-end | Generator → service → analyst dashboard | Docker + script | Injected attack appears in the review queue; analyst override (`APPROVE_OVERRIDE` / `CONFIRM_FRAUD`) is stored | FR-5 | M3 |
| Load | Latency and throughput | k6 + Grafana | P99 < 200 ms and 0 % errors at 500 RPS for 15 min | NFR-1, NFR-2 | M3 |
| Fault tolerance | ML engine stops under load | `docker stop` during a k6 run | 0 HTTP 5xx; breaker trips after ≤ 5 timeouts; service recovers after restart | NFR-3 | M3 |
| Explainability | Top-3 SHAP present and faithful | Automated check + manual review of 50 cases | 100 % of flagged decisions have top-3 features; 0 mismatches in the manual sample | NFR-5 | M3 |
| Quality experiment | B0–B2 vs M1–M2 | Replay of test split, SQL metrics | Criteria E1, E2, E4, E7 | — | M3 |

All automated tests run in GitHub Actions on every pull request.

## 8. Traceability: requirement → verification

| Requirement | Priority | Verified by | Evidence produced |
|-------------|----------|-------------|-------------------|
| FR-1 Ingestion API | Must | API validation, contract tests | CI log |
| FR-2 Sliding-window feature store | Must | Integration + benchmark, feature parity | CI log, benchmark output, parity report |
| FR-3 ML inference and risk scoring | Must | Offline model validation, quality experiment | Validation report, PR curve |
| FR-4 Tri-tier decision engine | Must | Unit tests, quality experiment | CI log, results table |
| FR-5 Analyst review queue and dashboard | Should | End-to-end test | Screenshot, stored override record |
| FR-6 Alerting and audit logging | Should | Audit logging test | Sample audit records |
| FR-7 Drift detection / shadow scoring | Could | Verified only if implemented | — |
| FR-8 Dynamic threshold recalibration | Won't | Out of scope | — |
| NFR-1 Latency SLA | Must | Load test | k6 summary, "load → P99" chart |
| NFR-2 Throughput ≥ 500 RPS | Must | 15-min load test | k6 summary |
| NFR-3 Idempotency and circuit breaking | Must | Idempotency test, fault test | CI log, Grafana screenshot of the outage |
| NFR-4 Data security | Must | Security scan | Scan report with 0 matches |
| NFR-5 Explainability | Should | Explainability check | Review sheet of 50 cases |

## 9. How the result is presented

Main results table (filled in April 2027):

| Configuration | PR-AUC | Recall @ FPR 1 % | Precision | FPR | Prevented loss, % | Review rate, % | P95 / P99, ms |
|---------------|--------|------------------|-----------|-----|-------------------|----------------|---------------|
| B1 Rules only | | | | | | | |
| B2 Logistic regression | | | | | | | |
| M1 LightGBM | | | | | | | |
| M2 Rules + LightGBM | | | | | | | |

Plus: a precision–recall curve per configuration, a "load (RPS) → P99 latency" chart, and the same table for IEEE-CIS. At the defense, the live demo shows normal traffic, an injected attack, an explained `DECLINE`, and stopping the ML engine without failed payments (E6).

## 10. Threats to validity

| Threat | Mitigation |
|--------|------------|
| Synthetic data (Sparkov) is too easy, results look too good | Acceptance (FR-3) on real IEEE-CIS; comparison repeated on both datasets (E7) |
| Leakage from the future | Time-based split; features use only past transactions |
| Rules or thresholds tuned on test data | All parameters fixed on train/validation before the test run |
| Latency depends on the laptop | Hardware recorded; configurations compared on the same machine |
| Absolute numbers will differ in a real bank | Claim only the relative comparison of approaches and the system's behaviour under load and failure |

## 11. Open points

| Item | Conflict | Proposed action |
|------|----------|-----------------|
| Behaviour on ML failure | SRS NFR-3: fail-open `ALLOW` after 5 timeouts > 250 ms. Architecture (`07_architecture.md`): decide by rules after a 30 ms timeout | Team decision needed; E6 and the fault test are written for the SRS and will be updated if it changes |
| FPR target | Proposal: < 2 %. SRS FR-3: ≤ 4.0 % | Treat 4.0 % as the pass/fail limit and 2 % as the target (E4) |
| Decision value in SRS FR-4 test | Table header lists `REVIEW`, test case uses `MANUAL_REVIEW` | Use `MANUAL_REVIEW` everywhere |

## 12. Schedule

| Milestone | Verification work |
|-----------|-------------------|
| M1 – Pre-defense 1 (24 Oct 2026) | This plan; datasets downloaded; split rules fixed |
| M2 – Pre-defense 2 | Unit, API, integration, idempotency, contract, parity and security tests in CI; rules baseline (B1) measured on validation |
| End of Feb 2027 | All configurations runnable; first load test |
| M3 – Final pre-defense (Apr 2027) | Final test-split runs, load and fault tests, explainability check, results table and charts |
