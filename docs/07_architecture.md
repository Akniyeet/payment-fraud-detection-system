# Architectural Description – Real-Time Payment Fraud Detection System

Sources:
- [`01_system_context.puml`](../architecture/01_system_context.puml)
- [`02_use_case.puml`](../architecture/02_use_case.puml)
- [`03_component.puml`](../architecture/03_component.puml)

## 1. Purpose and key requirements

The system scores each payment synchronously and returns APPROVE / REVIEW / DECLINE.

| ID    | Requirement                                                                 |
| ----- |---------------------------------------------------------------------------- |
| NFR-1 | p95 decision latency < 100 ms at ≥ 500 tx/s on a laptop (H3)                |
| NFR-2 | Degrade gracefully: if ML Service times out (~30 ms), fall back to rules    |
| NFR-3 | Idempotent scoring (repeated `tx_id` returns the same decision)             |
| NFR-4 | No training-serving skew: one Feature Builder (Go) for training and serving |
| NFR-5 | Whole stack starts with one `docker compose up`                             |
| FR-1  | Three decision modes: rules / ml / hybrid                                   |
| FR-2  | Analyst gets explanations (SHAP top-3 + LLM text, template fallback)        |
| FR-3  | Analyst review of REVIEW/DECLINE decisions is stored                        |

## 2. [System Context](../architecture/01_system_context.svg) (diagram 1)

The system is one black box. External parties:

- Payment Source: sends payments, receives a decision. In the project it is the Generator replaying Sparkov/IEEE-CIS; in production it would be a merchant or gateway.
- Fraud Analyst: watches the live stream, reads explanations, submits reviews.
- ML Engineer / Developer: edits rules (YAML), switches mode, delivers model artifacts, runs experiments.
- DevOps / Operator: deploys, watches metrics, runs k6 load tests, triggers demo attack scenarios.
- LLM Provider (cloud API or local Ollama): turns structured evidence into readable text. Optional; template fallback exists.
- Public Datasets: offline input for replay and training only. Labels never enter the live stream (stored separately in `ground_truth`).

## 3. [Use Cases](../architecture/02_use_case.svg) (diagram 2)

| Actor          | Use cases                                                                             |
| -------------- |-------------------------------------------------------------------------------------- |
| Payment Source | UC1 submit transaction, UC2 receive decision                                          |
| Fraud Analyst  | UC3 live stream, UC4 browse/filter, UC5 decision + explanation, UC6 review, UC7 stats |
| ML Engineer    | UC8 configure rules, UC9 switch mode, UC10 train/deploy model, UC11 experiments       |
| DevOps         | UC12 deploy, UC13 monitor, UC14 load/fault tests, UC15 trigger attack scenario        |
| LLM Provider   | participates in "Generate explanation"                                                |

Relations:
- UC1 "include" idempotency check, feature building, rule evaluation, UC2.
- "Get ML score + SHAP" "extend" UC1 only in ml/hybrid mode.
- "Fallback to rules" "extend" ML scoring when ML is unavailable.
- "Generate explanation" "extend" UC5 only for REVIEW/DECLINE.

## 4. [Components](../architecture/03_component.svg) (diagram 3, draft)

| Component      | Tech                                   | Responsibility                                                                                                                        |
| -------------- | -------------------------------------- |-------------------------------------------------------------------------------------------------------------------------------------- |
| Generator      | Go                                     | Replays dataset rows as payment requests; attack scenarios for demo                                                                   |
| Fraud Service  | Go                                     | Core. HTTP API, Idempotency Guard, Feature Builder, Rule Engine, Model Client, Decision Engine, Repository, SSE Hub, Metrics          |
| ML Service     | Python, FastAPI, LightGBM, SHAP        | `/predict` returns fraud probability + SHAP top-3                                                                                     |
| Explain Worker | LLM client + templates                 | Async explanation for REVIEW/DECLINE                                                                                                  |
| Dashboard      | React+TS or Go templates+HTMX          | Live feed, details, review, stats                                                                                                     |
| Redis          | Redis 7, go-redis, Lua script/pipeline | Sliding windows (`cust:{id}:tx` zset 24h), customer stats (hash, 90d), merchant/category sets, idempotency keys (`idem:{tx_id}`, 24h) |
| PostgreSQL     | PostgreSQL 16, pgx                     | `transactions`, `decisions`, `explanations`, `analyst_reviews`, `ground_truth`                                                        |
| Prometheus     | Prometheus                             | Scrapes `/metrics` of Fraud Service (Go client) and ML Service (Python client); p50/p95/p99 latency, decision and fallback counts     |
| Grafana        | Grafana                                | Dashboards on top of Prometheus                                                                                                       |

Sync path (latency budget): validate ≤1 ms → idempotency + features via Redis ≤3 ms → rules ≤1 ms → ML ≤30 ms → decision + DB write ≤10 ms.
Async path: Decision Engine → SSE Hub → Dashboard; Explain Worker → LLM → `explanations` table.

### Draft decisions / open points
- Explain Worker input channel is not specified in the spec: diagram shows polling PostgreSQL (simplest, no broker). Alternative: Redis stream or NATS.
- Dashboard framework still undecided (React+TS vs Go templates+HTMX).
- Single Fraud Service instance in compose; Fraud Service → ML Service over REST/JSON (ONNX-in-Go is an optional later improvement).
- Ports, exact payload schemas: defined in `api/openapi.yaml`, not duplicated here.

## 5. How to export images

### With PlantUML jar

Install: PlantUML jar (needs Java) or `sudo apt install plantuml` / `brew install plantuml`.

```sh
cd architecture
plantuml -tpng *.puml       # PNG
plantuml -tsvg *.puml       # SVG (better, scalable quality)
# For a large PNG, set the limit as an environment variable, and put skinparam dpi 200 in the file (it only affects PNG)
PLANTUML_LIMIT_SIZE=8192 plantuml -tpng *.puml 
```

### Using java directly

```sh
java -jar plantuml.jar -tsvg *.puml
```

### Online alternative

Paste the source into [`plantuml.com/plantuml`](https://plantuml.com/plantuml)