# Project Management, Milestones, Dependencies, and Risks

## 1. Project Milestones
- **Milestone 1 (Pre-Defense 1 — October 24):** Core analytical deliverables: problem proposal, relevance evidence, literature review matrix, requirements specification, draft architecture diagrams, and testing plan.
- **Milestone 2 (Pre-Defense 2):** Data preprocessing pipeline, baseline heuristic model, preliminary ML model training, and basic API endpoints.
- **Milestone 3 (Final Pre-Defense & Defense):** Real-time ML inference pipeline, integration with message broker/database, full verification report, and analyst dashboard prototype.

## 2. Technical Dependencies
- **Data Dependency:** IEEE-CIS Fraud Detection / Kaggle Credit Card dataset for baseline model validation.
- **Infrastructure Dependency:** Python 3.10+, FastAPI framework, Docker, and Git version control.
- **Compute Dependency:** GPU/CPU compute resources for model training and benchmark latency measurements.

## 3. Risk Management Matrix
| Risk Description | Probability | Impact | Mitigation Strategy |
| :--- | :---: | :---: | :--- |
| **Data Imbalance (rare fraud cases)** | High | High | Apply SMOTE, class-weighting, and evaluate models using PR-AUC / Recall rather than raw accuracy. |
| **High Inference Latency (>200ms)** | Medium | High | Optimize model serving using ONNX Runtime or lightweight tree ensembles (LightGBM/XGBoost). |
| **Data Drift (evolving fraud patterns)** | Medium | Medium | Design periodic retraining workflows and dynamic risk score thresholds. |
| **Scope Creep before Deadline** | High | Medium | Strictly adhere to MoSCoW prioritization outlined in YouTrack sprint tasks. |
