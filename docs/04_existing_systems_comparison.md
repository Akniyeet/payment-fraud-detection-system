# Existing Systems Comparison – Real-Time Payment Fraud Detection System

## 1. Purpose and method

This document compares existing payment fraud detection systems against one fixed set of criteria. The goal is to show what the market already solves, what it does not, and why the proposed system is still needed.

**Selection of systems.** Five systems were selected to cover the main types of solution on the market:

| # | System | Type | Why it was selected |
|---|--------|------|---------------------|
| S1 | Stripe Radar | ML built into a payment processor | Largest example of "fraud scoring as part of the gateway"; well-documented public docs |
| S2 | Sift (Payment Protection) | Independent SaaS, ML + workflows | Typical merchant-side SaaS with a score API and review queues |
| S3 | SEON | Independent SaaS, rules + "whitebox" ML | Emphasises transparent ML and custom rules; published pricing |
| S4 | Feedzai (RiskOps) | Enterprise platform for banks | Bank-side system, the target user group "financial institutions" from the proposal |
| S5 | Marble | Open-source decision engine | Only actively maintained open-source, self-hostable alternative found |

**Excluded:** *Amazon Fraud Detector*: AWS stopped accepting new customers on 7 November 2025 [E7], so it is no longer a realistic option for a new project. *FICO Falcon*, *Kount*, *Riskified* and *Signifyd* were considered, but their public technical documentation is too limited to verify most criteria, or they cover the same position as S1–S4 (chargeback-guarantee models such as Riskified/Signifyd work differently and are not directly comparable).

**Evidence rule.** Every cell is based on a public source listed in Section 5. Vendor marketing figures (e.g., "38 % less fraud") are marked as vendor claims and are **not** used to rank systems. If a property could not be confirmed from public sources, the cell says **"not public"** rather than guessing.

## 2. Comparison criteria

The criteria are taken from the project requirements (proposal §5–§7 and the architecture NFRs), so that the comparison answers the question "does an existing system already do what we need?".

| ID | Criterion | What is checked | Link to our requirements |
|----|-----------|-----------------|--------------------------|
| C1 | Deployment model | SaaS only / on-premise / self-hosted | Run locally with `docker compose`, data stays on own infrastructure (NFR-5 in `07_architecture.md`) |
| C2 | Detection approach | Rules, ML, or hybrid rules + ML | Hybrid decision modes (rules / ml / hybrid) (FR-1) |
| C3 | Real-time synchronous decision | Score returned in the request path, latency stated | p95 < 100 ms at ≥ 500 tx/s (NFR-1) |
| C4 | Decision outputs | Score only, or tiered decision (allow / review / block) | Three-tier decision `APPROVE` / `REVIEW` / `DECLINE` |
| C5 | Explainability | Per-transaction reasons visible to analysts | Feature attributions (SHAP top-3) for analysts |
| C6 | Analyst review tools | Review queue / case management | Analyst dashboard with review of flagged payments |
| C7 | Processor independence | Usable with any payment source, or only inside one processor | Generic scoring API for PSPs and banks |
| C8 | Cost and access | Published price, free tier, or quote only | Usable by a student team and small PSPs |
| C9 | Openness and reproducibility | Can the model/logic be inspected and benchmarked on a public dataset (IEEE-CIS) | Benchmarking ML vs. heuristic baseline is the research aim |

## 3. Comparison table

Legend: ✅ supported (documented), ◐ partially / with limits, ❌ not supported, **n/p** = not public.

| Criterion | S1 Stripe Radar | S2 Sift | S3 SEON | S4 Feedzai | S5 Marble |
|-----------|-----------------|---------|---------|------------|-----------|
| **C1 Deployment** | ❌ SaaS only, part of Stripe [E1] | ❌ SaaS only [E3] | ❌ SaaS only [E5] | ◐ Cloud or on-premise for banks; enterprise rollout [E6] | ✅ Self-hosted (Docker Compose) or hosted [E8] |
| **C2 Approach** | ✅ Hybrid: network ML score + rules (rules only in paid tiers) [E1][E2] | ✅ Hybrid: ML score + workflow rules on the score [E3] | ✅ Hybrid: custom rules + whitebox ML that suggests rules [E5] | ✅ Hybrid: rules + ML models [E6] | ◐ Rules only; AI helps write rules and investigate, no ML scoring [E8] |
| **C3 Real-time** | ✅ Evaluated in real time during payment; latency n/p [E1] | ✅ Real-time score via API; latency n/p [E3] | ✅ Real-time API; latency n/p [E5] | ✅ Real-time; "< 3 ms" is a third-party claim, not verified [E6] | ✅ Real-time monitoring API; no latency figures [E8] |
| **C4 Outputs** | ✅ Risk score + allow / review / block / 3DS [E1][E2] | ✅ Score 0–100 + accept / block / review queue via workflows [E3] | ✅ Score + approve / review / decline via rules [E5] | ✅ Risk score + decision n/p in detail [E6] | ✅ Rule-based decision outcomes [E8] |
| **C5 Explainability** | ✅ "Risk insights" list of risk factors per payment [E2] | ◐ Signals and data views; no dedicated reason-code docs found [E4] | ✅ Whitebox model, human-readable rules [E5] | ✅ "Whitebox" plain-text explanations (vendor claim) [E6] | ✅ Rules are fully readable (no ML to explain) [E8] |
| **C6 Analyst tools** | ✅ Review queue, related payments view [E2] | ✅ Review queues [E3] | ✅ Manual review dashboard [E5] | ✅ Case management [E6] | ✅ Case manager / investigation suite [E8] |
| **C7 Processor independence** | ❌ Built for payments processed on Stripe [E1] | ✅ Processor-agnostic API [E3] | ✅ Processor-agnostic API [E5] | ✅ Integrates with bank systems [E6] | ✅ Generic event API [E8] |
| **C8 Cost / access** | ◐ Published per-transaction fee (e.g., €0.05–0.09 per screened tx by tier) [E1] | ❌ Quote only, n/p | ◐ Published: from $699/month, 2,500 API calls [E5] | ❌ Quote only, high setup cost [E6] | ✅ Free open-source self-hosted edition; paid enterprise features [E8] |
| **C9 Openness / reproducibility** | ❌ Closed model trained on Stripe network data | ❌ Closed model, network data | ❌ Closed model | ❌ Closed model | ◐ Open code, but no ML model to benchmark |

## 4. Analysis

### 4.1 What existing systems already do well

- **Hybrid rules + ML is the industry standard.** S1–S4 all combine an ML risk score with rules on top of it. This supports the project decision to implement a hybrid mode instead of ML only (consistent with the literature review, `03_literature_review.md`).
- **Three-tier decisions are standard.** Every system returns something equivalent to allow / review / block. Our `APPROVE` / `REVIEW` / `DECLINE` output follows this convention, which makes the API familiar to integrators.
- **Analysts need explanations and a review queue.** All five systems have a review tool, and the market moved towards "whitebox" explanations (S1 risk insights, S3, S4). This confirms the requirement for SHAP-based reasons and an analyst dashboard.

### 4.2 Limitations that the proposed system addresses

1. **No system is both open and ML-based (C1 + C2 + C9).** The ML systems (S1–S4) are closed SaaS products; the only open, self-hostable system (S5) is rules-only. A team that wants to *inspect, reproduce and benchmark* an ML fraud model on its own infrastructure has no ready option.
2. **No published latency figures (C3).** All vendors claim "real-time", but none publishes a measurable latency under a stated load in its documentation. The project will publish p50/p95/p99 latency under a defined load (k6), which makes the claim verifiable.
3. **No public benchmark against a heuristic baseline (C9).** Vendor effectiveness figures (e.g., Stripe's "38 % fraud reduction", Feedzai's "62 % more fraud detected") are vendor claims on private data and cannot be reproduced. The project will compare rules-only vs. ML vs. hybrid on a public dataset (IEEE-CIS) with PR-AUC, recall at fixed FPR, and FPR.
4. **Cost and lock-in (C7, C8).** S1 only works inside Stripe; S2 and S4 are quote-only; S3 starts at several hundred USD per month. Small PSPs, local banks and student/research teams cannot easily use them for evaluation.
5. **Behaviour when the ML component fails is not documented.** None of the reviewed public docs describes fallback behaviour if the scoring model is unavailable. The project explicitly designs and tests a rules fallback on ML timeout.

### 4.3 What the project does **not** try to compete on

To keep the comparison fair: the commercial systems have advantages the project cannot match and does not target:

- **Consortium/network data**: S1 and S2 train on data from millions of businesses. The project uses only public datasets.
- **Production compliance**: PCI-DSS, certified bank integrations (S4). These are out of scope in the proposal.
- **Additional fraud types**: account takeover, AML, sanctions screening (S2, S4, S5). The project focuses on payment transaction fraud only.

### 4.4 Conclusion

The proposed system is not a replacement for commercial platforms. It is an **open, self-hosted, reproducible reference implementation** of a hybrid rules + ML scoring service, with measured latency, documented fallback and per-decision explanations. Its value is in the gap shown by C1 + C2 + C9: no reviewed system lets the user run, inspect and benchmark an ML fraud scorer against a heuristic baseline on their own infrastructure.

## 5. Evidence (sources)

Accessed October 2026.

| ID | Source | Used for |
|----|--------|----------|
| E1 | Stripe Radar docs and pricing: <https://docs.stripe.com/radar>, <https://stripe.com/radar/pricing> | S1: C1–C4, C7, C8 |
| E2 | Stripe Radar risk insights and rules: <https://docs.stripe.com/radar/reviews/risk-insights>, <https://stripe.com/radar/fraud-teams> | S1: C2, C4–C6 |
| E3 | Sift API overview and Score API: <https://developers.sift.com/docs/> | S2: C1–C4, C6, C7 |
| E4 | Sift blog "Sift scores": <https://sift.com/blog/sift-scores/> | S2: C5 |
| E5 | SEON pricing and product pages: <https://seon.io/pricing/>, Gartner reviews: <https://www.gartner.com/reviews/product/seon-172142893> | S3: C1–C8 |
| E6 | Feedzai RiskOps page: <https://feedzai.com/riskops>, Gartner profile: <https://www.gartner.com/reviews/market/online-fraud-detection/vendor/feedzai/product/feedzai-10262518> | S4: C1–C8 |
| E7 | AWS, Amazon Fraud Detector availability change: <https://docs.aws.amazon.com/frauddetector/latest/ug/frauddetector-availability-change.html> | Exclusion of Amazon Fraud Detector |
| E8 | Marble GitHub repository: <https://github.com/checkmarble/marble> | S5: C1–C9 |

**Limitations of this comparison.** Most sources are vendor documentation or marketing pages; no system was tested hands-on. Cells marked "n/p" may be supported but not publicly documented. Marble (S5) can be self-hosted and could be tested hands-on in a later iteration to confirm C3.
