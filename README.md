# Retail Customer Segmentation (Basic)

A data science project that segments retail customers into behaviorally distinct groups using K-Means clustering on RFM features derived from the UCI Online Retail Dataset. The goal is to replace uniform marketing strategies with targeted, evidence-based campaigns.

---

## Business Objective

> **"Group customers based on spending behavior"**

Retail businesses apply uniform marketing strategies across all customers, leading to inefficient spend and poor personalization. This project translates that problem into 3–5 specific analytical questions aligned to business operations and growth teams, then answers them with quantified evidence and next-step proposals.

**Key analytical questions:**
1. Which customers are high-value and at risk of churning?
2. How do purchase frequency and recency differ across segments?
3. What is the optimal number of segments (K) that balances interpretability and quality?
4. What targeted actions should each segment receive?
5. How stable are the clusters across different time periods and random seeds?

---

## Dataset Overview & Caveats

| Field | Detail |
|-------|--------|
| **Source** | UCI Online Retail Dataset |
| **Format** | `.xlsx` / `.csv` |
| **Rows (raw)** | ~541,909 transactions |
| **Key Fields** | `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country` |
| **Grain** | One row per line item per invoice |
| **Known Caveats** | ~25% of rows have no `CustomerID`; cancellations are prefixed with `C` in `InvoiceNo`; negative quantities and prices exist and must be filtered |

---

## Data Cleaning Summary

A documented cleaning pipeline with before/after row counts at every step:

| Step | Rows Remaining | Rows Removed |
|------|---------------|--------------|
| Raw load | 541,909 | — |
| Remove null CustomerIDs | 406,829 | 135,080 |
| Remove cancellations (InvoiceNo starts with `C`) | 397,924 | 8,905 |
| Remove negative quantities | 397,884 | 40 |
| Remove negative unit prices | 397,882 | 2 |
| Monetary outlier treatment (log-cap @ 99th pct) | 393,941 | 3,941 |

**Output:** `cleaned_data.parquet` — ~393K clean transaction rows.

---

## Feature Engineering

RFM features computed at the `CustomerID` level:

| Feature | Description | Transformation |
|---------|-------------|----------------|
| **Recency** | Days since last purchase (lower = better) | Log1p if skewness > 1.0 |
| **Frequency** | Number of unique invoices | Log1p (right-skewed) |
| **Monetary** | Total spend across dataset period | Log1p (right-skewed) |

Additional derived fields:
- **Total Spend** — `Quantity × UnitPrice` summed per customer
- **Average Order Value** — `Monetary / Frequency`
- **Purchase Frequency** — unique invoice count per customer

All features are scaled using **MinMaxScaler** (range [0, 1]) before clustering, since K-Means is a distance-based method sensitive to feature magnitude.

**Output:** `rfm_features.parquet`

---

## Clustering Approach

**Algorithm:** K-Means (scikit-learn)
**Candidate K values:** K=3, K=4, K=5
**Settings:** `n_init=20`, `max_iter=300`, `random_state=42`

### Model Comparison — K Values

| K | Silhouette Score | Inertia | Verdict |
|---|-----------------|---------|---------|
| K=3 | 0.39 | — | Underfits — segments too broad |
| **K=4** | **0.51** | — | **Recommended — best balance** |
| K=5 | 0.47 | — | Slight drop — marginal extra value |

Selection criterion: highest Silhouette Score + business interpretability.

---

## Cluster Profiles

Each cluster is assigned a named **business persona** with top distinguishing features and actionable recommendations:

| Cluster | Persona | Recency | Frequency | Monetary | Size |
|---------|---------|---------|-----------|----------|------|
| 0 | **Champions** | Low (recent) | High | High | ~18% |
| 1 | **Loyal Customers** | Low | High | Medium | ~32% |
| 2 | **At-Risk** | High | Medium | Medium | ~25% |
| 3 | **Bargain Hunters / Occasional** | High | Low | Low | ~25% |

### Business Recommendations per Persona

**Champions**
- Offer exclusive loyalty rewards and early-access sales
- Request product reviews and referrals
- Enroll in VIP tier program

**Loyal Customers**
- Send personalised upsell recommendations
- Offer subscription or loyalty points program
- Reward milestone purchases

**At-Risk**
- Trigger win-back email campaign with discount incentive
- Send re-engagement survey to identify pain points
- Offer limited-time bundle deal

**Bargain Hunters / Occasional**
- Target with clearance and seasonal promotions
- Use low-cost communication channels (email only)
- Monitor for upgrade potential over time

---

## Cluster Quality Evaluation

| Metric | Method | Target |
|--------|--------|--------|
| **Silhouette Score** | `sklearn.metrics.silhouette_score` | > 0.40 |
| **Stability Check** | Multi-seed bootstrapping (10 seeds) | `std(silhouette) < 0.02` |
| **Elbow Method** | Inertia vs K plot | Diminishing returns after K=4 |
| **Reproducibility** | Fixed `random_state=42` across all runs | Identical label arrays |

Stability is measured by running K-Means with 10 different random seeds and checking:
- Variance in Silhouette Score across trials
- Cluster size distribution deviation (%)
- Centroid position drift (Frobenius norm)

---

## Reproducible Pipeline

```
raw data (.xlsx/.csv)
    │
    ▼
data_ingestion.py       → load + validate schema
    │
    ▼
data_cleaning.py        → remove nulls, cancellations, outliers
    │
    ▼
feature_engineering.py  → compute RFM, log-transform, scale
    │
    ▼
clustering_engine.py    → fit KMeans for K=3,4,5
    │
    ▼
evaluation.py           → silhouette, stability, best K
    │
    ▼
persona_generator.py    → assign personas, generate recommendations
    │
    ▼
exportable tables / personas.json / dashboard
```

---

## Interactive Explorer

The dashboard allows business users to:

1. **Screen 1 — Landing Page:** View KPI overview (total customers, avg spend, avg frequency)
2. **Screen 2 — Select K:** Compare K=3/4/5 via Silhouette scores and Elbow chart; confirm cluster count
3. **Screen 3 — Cluster Visualization:** 2D RFM Scatter plot (Recency vs Monetary, colored by cluster) + Distribution bar chart
4. **Screen 4 — Cluster Detail:** Click any cluster to see avg RFM metrics, Silhouette score, stability status, cluster size
5. **Screen 5 — Recommendations:** View persona-specific business actions; export to PDF, send to CRM, or download customer list

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Python 3.10+ |
| Data Processing | Pandas, NumPy |
| Statistics | SciPy (distribution fitting, skewness/kurtosis) |
| ML / Clustering | Scikit-learn |
| Visualization | Matplotlib, Seaborn |
| Dashboard | Power BI / Tableau |
| API | FastAPI, Uvicorn |
| Frontend | React, Recharts |
| Environment | Jupyter Notebook, Conda / venv |
| Version Control | Git / GitHub |

---

## Project Structure

```
project/
├── notebooks/
│   ├── 01_data_ingestion.ipynb
│   ├── 02_cleaning.ipynb
│   ├── 03_feature_engineering.ipynb
│   ├── 04_clustering.ipynb
│   ├── 05_evaluation.ipynb
│   └── 06_persona_generation.ipynb
├── src/
│   ├── data_ingestion.py
│   ├── data_cleaning.py
│   ├── feature_engineering.py
│   ├── clustering_engine.py
│   ├── evaluation.py
│   ├── persona_generator.py
│   └── api_layer.py
├── data/
│   ├── raw/                  ← UCI dataset
│   └── processed/            ← cleaned_data.parquet, rfm_features.parquet
├── outputs/
│   ├── model_output.json
│   └── personas.json
├── frontend/                 ← React SPA
├── docs/
│   ├── consumer-flow.md
│   ├── data-level-flow.md
│   ├── high-level-flow.md
│   └── low-level-flow.md
├── Dockerfile
├── requirements.txt
└── README.md
```

---

## Deliverables

| # | Deliverable | Description |
|---|-------------|-------------|
| 1 | Data intake pack | Reproducible download script + data dictionary |
| 2 | Data quality report | Missingness, duplicates, outliers, leakage checks, before/after counts |
| 3 | Preprocessing spec | Scaling/normalization decisions with distance-based method rationale |
| 4 | Clustering experiment log | K=3, K=4, K=5 with quantitative comparison and qualitative profiling |
| 5 | Cluster quality evaluation | Silhouette score + stability checks across seeds/bootstraps |
| 6 | Cluster profiles | Named persona, top distinguishing features, 2 actionable recommendations each |
| 7 | Interactive explorer | Select cluster → see distributions, exemplars, recommended actions |
| 8 | Reproducible pipeline | `raw → features → clustering → profiles → exportable tables` |
| 9 | Final presentation | 8–10 slide readout summarizing data, method, findings, and recommendations |
| 10 | Executive summary | One-page brief with quantified evidence and next-step proposals |

---

## Limitations & Next Steps

**Current Limitations:**
- Pipeline is optimized for UCI dataset size (~550K rows); larger datasets would require Spark/Dask
- RFM features alone may miss product-category or geographic signals
- Static model — clusters are not automatically updated as new transactions arrive
- Dashboard is read-only; no real-time prediction for new customers without API integration

**Next Steps:**
- Integrate geographic and product-category features to enrich segments
- Schedule automated retraining via Apache Airflow DAG
- Deploy MLflow for model versioning and experiment tracking
- Connect Power BI to live DirectQuery for real-time cluster updates
- Extend API with `/predict` endpoint for CRM integration to score new customers on the fly

---

## Documentation

| File | Description |
|------|-------------|
| [consumer-flow.md](consumer-flow.md) | End-to-end business user interaction flow across 5 screens |
| [data-level-flow.md](data-level-flow.md) | Level 0 context DFD and Level 1 detailed data flow diagram |
| [high-level-flow.md](high-level-flow.md) | System architecture, components, tech stack, scalability, NFRs |
| [low-level-flow.md](low-level-flow.md) | Module-level design, class signatures, error handling, validation checklist |
