# High-Level Design (HLD)
## Retail Customer Segmentation

**Project:** Retail Customer Segmentation (Basic)
**Dataset:** UCI Online Retail Dataset
**ML Core:** RFM Feature Engineering + K-Means Clustering
**Stack:** Python | Scikit-learn | FastAPI | React | Power BI
**Version:** 1.0 | Enterprise-Grade

---

## 1.1 System Overview

The Retail Customer Segmentation system ingests transactional data from the UCI Online Retail Dataset, applies data quality pipelines, engineers RFM (Recency, Frequency, Monetary) behavioral features, and runs K-Means clustering to identify distinct customer personas. The output is consumed through an interactive dashboard (Power BI / Tableau) and optionally served via a FastAPI + React web application for real-time segment exploration.

The system is architected in two deployment tiers:

- **Notebook Tier:** Jupyter-based exploratory pipeline for data scientists — rapid iteration, visualization, and model tuning.
- **Production Tier:** Modular Python package + FastAPI REST backend + React SPA frontend — suitable for business deployment and CI/CD integration.

### Core Business Problem

Retail businesses apply uniform marketing strategies across all customers, leading to inefficient spend and poor personalization. This system identifies 3–5 behaviorally distinct customer segments enabling targeted campaigns, loyalty programs, and churn prevention strategies.

---

## 1.2 Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  SYSTEM ARCHITECTURE — RETAIL SEGMENTATION                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────────────────────────────┐        │
│  │  DATA SOURCES   │    │           PROCESSING LAYER              │        │
│  ├─────────────────┤    ├─────────────────────────────────────────┤        │
│  │  UCI Online     │───►│  data_ingestion.py                      │        │
│  │  Retail Dataset │    │    └─► data_cleaning.py                 │        │
│  │  (.xlsx / .csv) │    │         └─► feature_engineering.py      │        │
│  └─────────────────┘    │              └─► clustering_engine.py   │        │
│                         │                   └─► evaluation.py     │        │
│                         │                        └─► persona_gen  │        │
│                         └─────────────────────────────────────────┘        │
│                                           │                                 │
│                         ┌─────────────────▼─────────────────────┐          │
│                         │              DATA STORES               │          │
│                         │  ┌──────────────────────────────────┐  │          │
│                         │  │  cleaned_data.parquet            │  │          │
│                         │  │  rfm_features.parquet            │  │          │
│                         │  │  model_output.json               │  │          │
│                         │  │  cluster_personas.json           │  │          │
│                         │  └──────────────────────────────────┘  │          │
│                         └─────────────────────────────────────── ┘          │
│                                           │                                 │
│                    ┌──────────────────────▼──────────────────────┐          │
│                    │                SERVING LAYER                 │          │
│                    │  ┌─────────────────────┐  ┌──────────────────────────┐│
│                    │  │  FastAPI Backend     │  │  Power BI / Tableau      ││
│                    │  │  /segment            │  │  Dashboard               ││
│                    │  │  /metrics            │  │  - RFM Scatter           ││
│                    │  │  /personas           │  │  - Cluster Distribution  ││
│                    │  └──────────┬───────────┘  │  - Persona Cards         ││
│                    │             │               └──────────────────────────┘│
│                    │  ┌──────────▼───────────┐                              │
│                    │  │  React SPA Frontend  │                              │
│                    │  │  /dashboard          │                              │
│                    │  │  /explorer           │                              │
│                    │  └──────────────────────┘                              │
│                    └──────────────────────────────────────────────┘         │
│                                                                             │
│                              ▲            ▲                                 │
│                        Business      Data Scientist                         │
│                          User                                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1.3 Major Components

| Component | Module | Responsibility | Technology |
|-----------|--------|----------------|------------|
| Data Ingestion | `data_ingestion.py` | Load raw `.xlsx`/`.csv`, validate schema, log row counts | Pandas, openpyxl |
| Data Cleaning | `data_cleaning.py` | Remove nulls, cancellations, negative qty, outliers | Pandas, SciPy |
| Feature Engineering | `feature_engineering.py` | Compute RFM scores, log-transform, scale features | Pandas, NumPy, Sklearn |
| Clustering Engine | `clustering_engine.py` | K-Means for K=3,4,5; inertia elbow method | Scikit-learn |
| Evaluation | `evaluation.py` | Silhouette score, stability across seeds, elbow plots | Scikit-learn, Matplotlib |
| Persona Generator | `persona_generator.py` | Map clusters to business personas, generate insights | Pandas, JSON |
| API Layer | `api_layer.py` | REST endpoints for segmentation results and metrics | FastAPI, Uvicorn |
| Dashboard | Power BI / Tableau | Interactive cluster explorer, persona cards, KPIs | Power BI / Tableau |
| React Frontend | `src/App.jsx` | SPA consuming FastAPI; segment explorer, charts | React, Axios, Recharts |

---

## 1.4 Technology Stack Layering

| Layer | Technologies |
|-------|-------------|
| Presentation | React.js, Recharts, Power BI, Tableau |
| API / Service | FastAPI, Uvicorn, Pydantic, CORS |
| ML / Analytics | Scikit-learn, SciPy, NumPy, Pandas |
| Feature Store | Parquet files (local / S3-compatible) |
| Data Ingestion | Pandas, openpyxl, Pathlib |
| Infrastructure | Docker (optional), GitHub Actions CI |
| Environment | Python 3.10+, Conda / venv, Jupyter |

---

## 1.5 Deployment Architecture

### Notebook Tier (Development / Exploration)

```
project/
  notebooks/
    01_data_ingestion.ipynb
    02_cleaning.ipynb
    03_feature_engineering.ipynb
    04_clustering.ipynb
    05_evaluation.ipynb
    06_persona_generation.ipynb
  data/
    raw/        ← UCI dataset
    processed/  ← cleaned_data.parquet, rfm_features.parquet
  outputs/      ← cluster_results.json, personas.json
```

### Production Tier (Deployment)

```
project/
  src/
    data_ingestion.py
    data_cleaning.py
    feature_engineering.py
    clustering_engine.py
    evaluation.py
    persona_generator.py
    api_layer.py          ← FastAPI app
  frontend/               ← React SPA
  Dockerfile              ← Container packaging
  requirements.txt
  .github/workflows/ci.yml
```

---

## 1.6 Scalability Considerations

| Concern | Current Approach | Scale-Out Strategy |
|---------|-----------------|-------------------|
| Data Volume | Pandas in-memory (UCI ~550K rows) | Replace Pandas with PySpark or Dask for 10M+ rows |
| Model Retraining | Manual notebook execution | Apache Airflow DAG or cron-based scheduled job |
| API Throughput | Single Uvicorn worker | Gunicorn multi-worker + Redis caching for segment results |
| Storage | Local Parquet files | AWS S3 + Delta Lake or Azure Blob Storage |
| Model Registry | Local JSON files | MLflow Tracking Server + Model Registry |
| Dashboard | Static Power BI file | Power BI Service with live DirectQuery or incremental refresh |

---

## 1.7 Non-Functional Requirements

| NFR Category | Requirement | Target Metric |
|-------------|-------------|---------------|
| Performance | Full pipeline execution time | < 5 minutes on UCI dataset (~550K rows) |
| Latency | API response time per endpoint | < 500ms p95 for `/segment` and `/personas` |
| Reproducibility | Identical results given same seed | `random_state=42` globally; seeded NumPy + Sklearn |
| Accuracy | Clustering quality threshold | Silhouette Score > 0.40 on optimal K |
| Stability | Model consistency across seeds | < 5% variance in cluster sizes across 10 seed runs |
| Maintainability | Code coverage and modularity | Each module < 200 LOC; pytest coverage > 80% |
| Observability | Logging and audit trail | Python logging to file + console; structured JSON logs |
| Portability | Environment compatibility | Dockerfile + requirements.txt for any OS deployment |
