# Data Flow Diagram (DFD)
## Retail Customer Segmentation

---

## 4.1 Level 0 DFD — Context Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        LEVEL 0 — CONTEXT DFD                            │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐            ┌────────────────────────────────────┐     │
│  │              │  Raw Data  │                                    │     │
│  │  UCI Online  │──────────► │                                    │     │
│  │  Retail DB   │            │                                    │     │
│  └──────────────┘            │    RETAIL SEGMENTATION SYSTEM      │     │
│                              │                                    │     │
│  ┌──────────────┐            │                                    │     │
│  │              │  K Value   │                                    │     │
│  │   Business   │──────────► │                                    │     │
│  │   User       │  Config    │                                    │     │
│  │              │◄───────────│                                    │     │
│  └──────────────┘  Segment   └────────────────────────────────────┘     │
│                    Reports                    │                          │
│                    Personas        Segmentation│                         │
│                    Charts          Results    ▼                          │
│                              ┌───────────────────────┐                  │
│                              │   DASHBOARD /         │                  │
│                              │   API CONSUMERS       │                  │
│                              └───────────────────────┘                  │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Level 1 DFD — Detailed Process Flow

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                         LEVEL 1 — DETAILED DFD                             ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  EXTERNAL ENTITIES            PROCESSES                  DATA STORES         ║
║  ═════════════════            ═════════                  ═══════════         ║
║                                                                              ║
║  ┌─────────────┐                                                             ║
║  │ UCI Online  │                                                             ║
║  │ Retail DB   │──── raw_transactions ────► ┌────────────────────┐          ║
║  │ (.xlsx/.csv)│                            │  P1: DATA          │          ║
║  └─────────────┘                            │  INGESTION         │          ║
║                                             │  - Load file       │          ║
║                                             │  - Schema check    │          ║
║                                             │  - Log row count   │          ║
║                                             └────────┬───────────┘          ║
║                                                      │ validated_df          ║
║                                                      ▼                       ║
║                                             ┌────────────────────┐          ║
║                                             │  P2: DATA          │          ║
║                                             │  CLEANING          │──► DS1: cleaned_data.parquet
║                                             │  - Remove nulls    │          ║
║                                             │  - Remove C-invoices│         ║
║                                             │  - Remove neg qty  │          ║
║                                             │  - Outlier treat   │          ║
║                                             └────────┬───────────┘          ║
║                                                      │ clean_df + audit_log  ║
║                                                      ▼                       ║
║                                             ┌────────────────────┐          ║
║                                             │  P3: FEATURE       │          ║
║                                             │  ENGINEERING       │──► DS2: rfm_features.parquet
║                                             │  - Compute RFM     │          ║
║                                             │  - Log transform   │          ║
║                                             │  - MinMax scaling  │          ║
║                                             └────────┬───────────┘          ║
║                                                      │ X_scaled (N×3)        ║
║  ┌────────────┐                                      ▼                       ║
║  │ Business   │──── K config ──────────► ┌────────────────────┐             ║
║  │ User       │                          │  P4: CLUSTERING    │             ║
║  └────────────┘                          │  ENGINE            │──► DS3: model_output.json
║                                          │  - KMeans K=3,4,5  │             ║
║                                          │  - Inertia/elbow   │             ║
║                                          └────────┬───────────┘             ║
║                                                   │ labels + centroids       ║
║                                                   ▼                          ║
║                                          ┌────────────────────┐             ║
║                                          │  P5: EVALUATION    │             ║
║                                          │  - Silhouette score│             ║
║                                          │  - Stability test  │             ║
║                                          │  - Best K select   │             ║
║                                          └────────┬───────────┘             ║
║                                                   │ metrics_report           ║
║                                                   ▼                          ║
║                                          ┌────────────────────┐             ║
║                                          │  P6: PERSONA       │             ║
║                                          │  GENERATION        │──► DS4: personas.json
║                                          │  - Cluster stats   │             ║
║                                          │  - Persona mapping │             ║
║                                          │  - Recs generation │             ║
║                                          └────────┬───────────┘             ║
║                                                   │ personas_json            ║
║                        ┌──────────────────────────┼──────────────────────┐  ║
║                        ▼                          ▼                      ▼  ║
║             ┌───────────────────┐  ┌───────────────────┐  ┌─────────────────┐║
║             │  P7a: FastAPI     │  │  P7b: Power BI /  │  │  P7c: Export /  │║
║             │  REST API         │  │  Tableau Dashboard│  │  CRM Integration│║
║             │  /segment         │  │  - Scatter plot   │  │  - CSV export   │║
║             │  /personas        │  │  - Persona cards  │  │  - Segment IDs  │║
║             │  /metrics         │  │  - KPI tiles      │  │                 │║
║             └─────────┬─────────┘  └────────┬──────────┘  └─────────────────┘║
║                       │                     │                                 ║
║                       └─────────────────────┘                                ║
║                                   │                                           ║
║                                   ▼                                           ║
║                        ┌──────────────────────┐                              ║
║                        │    BUSINESS USER      │                              ║
║                        │  Views segments,      │                              ║
║                        │  takes action         │                              ║
║                        └──────────────────────┘                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## 4.3 Data Store Reference

| Store ID | Name | Format | Contents | Produced By |
|----------|------|--------|----------|-------------|
| DS1 | cleaned_data | Parquet | Cleaned transactions: ~393K rows post-filtering | P2: Data Cleaning |
| DS2 | rfm_features | Parquet | CustomerID + Recency, Frequency, Monetary (raw + scaled) | P3: Feature Engineering |
| DS3 | model_output | JSON + PKL | KMeans model (serialized), cluster labels, inertia dict | P4: Clustering Engine |
| DS4 | personas | JSON | Cluster → persona mapping + recommendations + stats | P6: Persona Generator |

---

## Process Summary

| Process | Module | Input | Output |
|---------|--------|-------|--------|
| P1: Data Ingestion | `data_ingestion.py` | Raw `.xlsx`/`.csv` | validated_df |
| P2: Data Cleaning | `data_cleaning.py` | validated_df | DS1: cleaned_data.parquet |
| P3: Feature Engineering | `feature_engineering.py` | cleaned_df | DS2: rfm_features.parquet |
| P4: Clustering Engine | `clustering_engine.py` | X_scaled + K config | DS3: model_output.json |
| P5: Evaluation | `evaluation.py` | labels + centroids | metrics_report |
| P6: Persona Generation | `persona_generator.py` | metrics_report | DS4: personas.json |
| P7a: FastAPI REST API | `api_layer.py` | personas_json | `/segment`, `/personas`, `/metrics` |
| P7b: Dashboard | Power BI / Tableau | personas_json | RFM Scatter, Cluster Distribution, Persona Cards |
| P7c: Export / CRM | CRM Integration | personas_json | CSV export, Segment IDs |
