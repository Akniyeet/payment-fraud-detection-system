# Scoring

1-5 per criterion, 5 = best for this project. Total = weighted average (weights: latency 25, isolation 20, consistency 20, footprint 15, delivery risk 15, ecosystem 5). All scores were estimated. Triton, BentoML and Go+cgo ratings are not checked against their docs.

Criterion names are adapted per decision (stated under each table). Abbreviations: Lat = latency, Iso = isolation, Cons = consistency, Foot = footprint, Risk = delivery risk (5 = low risk), Eco = ecosystem.

# Decision 1a: Primary store

## Table

Iso = durability/ACID, Cons = relational fit (UNIQUE tx_id, joins, updates, SQL time splits)

| Option        | Lat   | Iso   | Cons  | Foot  | Risk  | Eco   | Total |
| ------------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| PostgreSQL    |     4 | 5     | 5     | 4     | 5     | 5     | 4.60  |
| MySQL/MariaDB |     4 | 5     | 4     | 4     | 4     | 5     | 4.25  |
| MongoDB       |     4 | 4     | 2     | 3     | 3     | 4     | 3.30  |
| ClickHouse    |     2 | 2     | 2     | 3     | 2     | 3     | 2.20  |

## Why PostgreSQL?

It matches the data (relational: transactions, decisions, reviews, ground_truth) and has real ACID, so `UNIQUE(tx_id)` backs up Redis idempotency. Its insert (~1-3 ms) fits the 10 ms decision + DB budget, and SQL gives clean time-based train/val/test splits. MySQL is nearly equal but has no advantage here. MongoDB loses on relational fit. ClickHouse is built for bulk analytics, not per-row inserts and row updates (analyst reviews).

# Decision 1b: Hot feature store

## Table

Iso = atomicity + survives restart/shared across instances, Cons = atomic read-update of rolling windows

| Option            | Lat   | Iso   | Cons  | Foot  | Risk  | Eco   | Total |
| ----------------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| Redis             | 5     | 4     | 5     | 4     | 5     | 5     | 4.65  |
| Dragonfly         | 5     | 4     | 4     | 4     | 3     | 3     | 4.05  |
| PostgreSQL only   | 2     | 5     | 4     | 4     | 4     | 5     | 3.75  |
| In-process Go map | 5     | 1     | 3     | 5     | 4     | 4     | 3.60  |
| Memcached         | 5     | 2     | 1     | 5     | 3     | 4     | 3.25  |

## Why Redis?

zsets plus a Lua script give atomic rolling-window read+update (tx_count_1h/24h, sum_24h) in sub-ms, and state survives a service restart. Memcached lacks those structures, so windows would be computed client-side. The in-process map is fast but loses state on restart and ties it to one process. Caveat: PostgreSQL-only scores 3.75 only because it is otherwise capable. I expect it to break the ≤3 ms Redis budget at 500 tx/s, which makes it a latency-gate failure that the weighted total hides. The benchmark should confirm this.

# Decision 2: Backend

## Table

Lat = tail latency, Iso = concurrency + fault isolation (timeouts, fallback), Cons = train/serve feature parity, Foot = RAM on a shared laptop

| Option               | Lat   | Iso   | Cons  | Foot  | Risk  | Eco   | Total |
| -------------------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| Go                   | 4     | 5     | 4     | 5     | 4     | 4     | 4.35  |
| Rust (Axum)          | 5     | 5     | 4     | 5     | 2     | 3     | 4.25  |
| Spring Boot          | 3     | 4     | 4     | 2     | 3     | 5     | 3.35  |
| FastAPI (all-Python) | 2     | 2     | 5     | 3     | 5     | 5     | 3.35  |
| Node.js (Fastify)    | 3     | 3     | 3     | 3     | 4     | 4     | 3.20  |

## Why not Rust?

It is nearly tied (the gap is 0.10). It is faster, but latency isn't the bottleneck (ML is ≤30 ms of the budget). Rust would win only if latency weighed more than was estimated, furthermore it requires to spend more time on learning new language.

## Why not FastAPI-only?

It solves parity, but scoring and ML would share one process, and the GIL hurts CPU-bound rules.

## Spring Boot

Heaviest RAM and JVM warmup, with moderate dev effort.

## Why Go?

* Goroutines plus `context.WithTimeout` give the ML timeout and rules fallback in a few lines.
* Services take tens of MB, which matters with ~6 other containers.
* It has the best delivery risk among the low-footprint options.
* Its one weak spot is parity: the Go Feature Builder is separate from the Python training code, so the spec's 1000-tx parity check is required.

# Decision 3: ML serving

## Table

Iso = independent failure from the Fraud Service, Cons = same artifact as training and in-process SHAP, Lat includes SHAP top-3 inside the 30 ms budget

| Option                      | Lat   | Iso   | Cons  | Foot  | Risk  | Eco   | Total |
| --------------------------- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| FastAPI + LightGBM (native) | 4     | 5     | 5     | 4     | 5     | 5     | 4.60  |
| FastAPI + ONNX Runtime      | 4     | 5     | 2     | 4     | 3     | 4     | 3.65  |
| BentoML                     | 3     | 5     | 4     | 3     | 3     | 4     | 3.65  |
| Go + cgo LightGBM C API     | 5     | 1     | 4     | 5     | 2     | 2     | 3.40  |
| ONNX Runtime in Go          | 5     | 1     | 2     | 5     | 2     | 3     | 3.05  |
| Triton                      | 3     | 5     | 2     | 1     | 2     | 4     | 2.80  |

## Alternative options

Why not the in-process Go options: they save the network hop (~1-3 ms, estimated) but share a process with the Fraud Service. A cgo crash takes down scoring, and ONNX might lose SHAP. Robustness check: with isolation weighted 0 and latency raised to 45, native still wins (4.40 vs 4.20 for Go+cgo). Triton's batching trades latency for throughput, and BentoML is still Python without adding anything.

## Why native FastAPI?

* It loads the exact trained LightGBM model, so there is no conversion step.
* SHAP runs in-process, so the 30 ms budget covers the whole call.
* It's a separate process, so ML can fail without taking scoring down, and the fallback and fault-tolerance tests mean something.
* Its cost is an HTTP hop plus the GIL, a few ms against a 30 ms budget, mitigated with multiple workers.

# Final Results

- Main storage: PostgreSQL
- Additional storage: Redis
- Backend: Go
- ML serving: FastAPI + LightGBM (native)