# Low-Level Design (LLD)
## Retail Customer Segmentation

---

## LLD Process Flow Diagram

```
Start: Load Configuration (file_path, file_format)
                    │
                    ▼
    ┌───────────────────────────────┐
    │  Module 1: Data Ingestion     │
    │  → Load .xlsx / .csv          │
    │  → Validate Required Columns  │
    │  → Log Row Count              │
    └───────────────┬───────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │  Module 2: Data Cleaning Pipeline     │
    │  → Remove Null CustomerIDs            │
    │  → Remove Cancellations               │
    │    (InvoiceNo starts with C)          │
    │  → Remove Negative Quantity           │
    │  → Remove Negative UnitPrice          │
    │  → Compute TotalPrice                 │
    │  → Treat Monetary Outliers            │
    │    (IQR / log_cap)                    │
    └───────────────┬───────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │  Module 3: Feature Engineering        │
    │  → Compute RFM                        │
    │    (Recency, Frequency, Monetary)     │
    │  → Apply log1p Transform              │
    │  → Scale Features (MinMax / Standard) │
    │  → Generate Feature Matrix            │
    │    (N_customers × 3)                  │
    └───────────────┬───────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │  Module 4: Clustering Engine          │
    │  → Fit KMeans (K=3,4,5)              │
    │  → Store Labels + Centroids           │
    │  → Compute Inertia (Elbow Data)       │
    └───────────────┬───────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │  Module 5: Model Evaluation           │
    │  → Compute Silhouette Scores          │
    │  → Select Best K                      │
    │  → Stability Check (Multi-Seed)       │
    └───────────────┬───────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │  Module 6: Persona Generator          │
    │  → Compute Cluster Statistics         │
    │  → Assign Business Personas           │
    │  → Generate Recommendations           │
    │  → Export personas.json               │
    └───────────────┬───────────────────────┘
                    │
                    ▼
    ┌───────────────────────────────────────┐
    │  Module 7: API Layer (FastAPI)        │
    │  → Load Model Artifacts on Startup    │
    │  → /segment Endpoint                  │
    │  → /personas Endpoint                 │
    │  → /metrics Endpoint                  │
    │  → /predict Endpoint                  │
    └───────────────┬───────────────────────┘
                    │
                    ▼
    End: Business User Consumes Segmentation Results
```

---

## 2.1 Module: `data_ingestion.py`

**Responsibility:** Loads raw UCI Online Retail data (`.xlsx` or `.csv`), validates schema integrity, logs initial row counts, and returns a raw DataFrame for downstream cleaning.

### Class & Function Signatures

```python
class DataIngestionConfig:
    file_path: str
    file_format: Literal['xlsx', 'csv'] = 'xlsx'
    sheet_name: str                      = 'Online Retail'
    encoding: str                        = 'latin-1'

class DataIngestionPipeline:
    def __init__(self, config: DataIngestionConfig) -> None

    def load_data(self) -> pd.DataFrame:
        # Loads file using pandas read_excel or read_csv
        # Validates presence of required columns
        # Logs: shape, dtypes, null counts
        # Returns: raw DataFrame

    def validate_schema(self, df: pd.DataFrame) -> bool:
        # Checks for required columns:
        # ['InvoiceNo','StockCode','Description','Quantity',
        #  'InvoiceDate','UnitPrice','CustomerID','Country']
        # Raises SchemaValidationError if missing

    def log_row_count(self, df: pd.DataFrame, stage: str) -> None:
        # Emits: logger.info(f'[{stage}] Rows: {len(df):,}')

REQUIRED_COLUMNS = [
    'InvoiceNo', 'StockCode', 'Description',
    'Quantity', 'InvoiceDate', 'UnitPrice',
    'CustomerID', 'Country'
]
```

### Data Structures

- **Input:** `.xlsx` / `.csv` file on disk
- **Output:** `pd.DataFrame` with 8 standard columns, raw dtypes preserved
- **Logging:** JSON-structured log entries with timestamp, stage, row_count

### Error Handling

| Error | Handling |
|-------|----------|
| `FileNotFoundError` | Raised with file path in message |
| `SchemaValidationError` (custom) | Raised listing missing columns |
| `EmptyDataError` | Raised if loaded DataFrame has 0 rows |

---

## 2.2 Module: `data_cleaning.py`

**Responsibility:** Applies all data quality transformations: null removal, cancellation filtering, negative quantity/price removal, outlier treatment on Monetary values using IQR or log-cap. Documents before/after row counts at each step.

### Function Signatures

```python
class DataCleaningPipeline:
    def __init__(self, df: pd.DataFrame) -> None
    self.audit_log: List[Dict] = []  # tracks row counts per step

    def remove_null_customer_ids(self) -> 'DataCleaningPipeline':
        # df.dropna(subset=['CustomerID'])

    def remove_cancellations(self) -> 'DataCleaningPipeline':
        # mask = ~df['InvoiceNo'].str.startswith('C')

    def remove_negative_quantities(self) -> 'DataCleaningPipeline':
        # df = df[df['Quantity'] > 0]

    def remove_negative_prices(self) -> 'DataCleaningPipeline':
        # df = df[df['UnitPrice'] > 0]

    def compute_total_price(self) -> 'DataCleaningPipeline':
        # df['TotalPrice'] = df['Quantity'] * df['UnitPrice']

    def treat_monetary_outliers(
        self,
        method: Literal['iqr', 'log_cap'] = 'log_cap',
        cap_percentile: float = 0.99
    ) -> 'DataCleaningPipeline':
        # IQR method: cap at Q3 + 1.5*IQR
        # log_cap: np.log1p transform then cap at cap_percentile

    def get_cleaned_data(self) -> pd.DataFrame
    def get_audit_report(self) -> pd.DataFrame  # before/after table

# Chaining pattern:
# pipeline = DataCleaningPipeline(raw_df)
#   .remove_null_customer_ids()
#   .remove_cancellations()
#   .remove_negative_quantities()
#   .remove_negative_prices()
#   .compute_total_price()
#   .treat_monetary_outliers(method='log_cap')
```

### Before/After Audit Log Format

```python
[
    {'step': 'raw_load',              'rows': 541909},
    {'step': 'remove_null_customer',  'rows': 406829, 'removed': 135080},
    {'step': 'remove_cancellations',  'rows': 397924, 'removed': 8905},
    {'step': 'remove_negative_qty',   'rows': 397884, 'removed': 40},
    {'step': 'remove_negative_price', 'rows': 397882, 'removed': 2},
    {'step': 'outlier_treatment',     'rows': 393941, 'removed': 3941}
]
```

---

## 2.3 Module: `feature_engineering.py`

**Responsibility:** Computes RFM features at the CustomerID level. Applies log1p transformation to handle monetary skewness. Scales features using MinMaxScaler or StandardScaler. Outputs feature matrix ready for clustering.

### RFM Computation Logic

```python
SNAPSHOT_DATE = df['InvoiceDate'].max() + timedelta(days=1)

rfm = df.groupby('CustomerID').agg(
    Recency   = ('InvoiceDate', lambda x: (SNAPSHOT_DATE - x.max()).days),
    Frequency = ('InvoiceNo',   'nunique'),
    Monetary  = ('TotalPrice',  'sum')
).reset_index()

# Recency:  lower = better (more recent purchase)
# Frequency: higher = more transactions
# Monetary: total spend in dataset period
```

### Log Transformation (Skewness Handling)

```python
rfm['Monetary_log']  = np.log1p(rfm['Monetary'])
rfm['Frequency_log'] = np.log1p(rfm['Frequency'])

# Recency is often less skewed; transform only if skewness > 1.0
if rfm['Recency'].skew() > 1.0:
    rfm['Recency_log'] = np.log1p(rfm['Recency'])
```

### Scaling Logic

```python
class FeatureEngineeringPipeline:
    def __init__(
        self,
        scaler_type: Literal['minmax', 'standard'] = 'minmax'
    ) -> None:
        self.scaler = MinMaxScaler() if scaler_type == 'minmax' \
                      else StandardScaler()

    def compute_rfm(self, df: pd.DataFrame) -> pd.DataFrame
    def apply_log_transform(self, rfm: pd.DataFrame) -> pd.DataFrame
    def scale_features(
        self,
        rfm: pd.DataFrame,
        feature_cols: List[str]
    ) -> np.ndarray
    def get_feature_matrix(self) -> np.ndarray  # shape: (N_customers, 3)
    def get_rfm_df(self) -> pd.DataFrame         # with CustomerID index
    def save_to_parquet(self, path: str) -> None
```

### Data Structures

- **Input:** cleaned `pd.DataFrame` with `InvoiceDate`, `InvoiceNo`, `TotalPrice`, `CustomerID`
- **Intermediate:** `rfm` DataFrame — columns: `CustomerID`, `Recency`, `Frequency`, `Monetary`
- **Output:** `np.ndarray` shape `(N_customers, 3)` — scaled, log-transformed feature matrix

---

## 2.4 Module: `clustering_engine.py`

### K-Means Implementation

```python
class ClusteringEngine:
    def __init__(
        self,
        k_range: List[int] = [3, 4, 5],
        random_state: int   = 42,
        n_init: int         = 20,
        max_iter: int       = 300
    ) -> None:
        self.models:   Dict[int, KMeans]    = {}
        self.labels:   Dict[int, np.ndarray] = {}
        self.inertias: Dict[int, float]      = {}

    def fit_all(self, X: np.ndarray) -> 'ClusteringEngine':
        for k in self.k_range:
            km = KMeans(n_clusters=k, n_init=self.n_init,
                        max_iter=self.max_iter, random_state=self.random_state)
            km.fit(X)
            self.models[k]   = km
            self.labels[k]   = km.labels_
            self.inertias[k] = km.inertia_

    def get_labels(self, k: int) -> np.ndarray
    def get_centroids(self, k: int) -> np.ndarray   # shape: (k, 3)
    def get_inertia_series(self) -> Dict[int, float]
    def predict_new(self, X_new: np.ndarray, k: int) -> np.ndarray
```

### Elbow Method Logic

```python
def compute_elbow_scores(inertias: Dict[int, float]) -> Dict:
    ks     = sorted(inertias.keys())
    deltas = [inertias[ks[i]] - inertias[ks[i+1]] for i in range(len(ks)-1)]
    return {'k': ks, 'inertia': list(inertias.values()), 'delta': deltas}
```

---

## 2.5 Module: `evaluation.py`

### Silhouette Score Calculation

```python
class ModelEvaluator:
    def compute_silhouette(self, X: np.ndarray, labels: np.ndarray) -> float:
        return silhouette_score(X, labels, metric='euclidean')

    def compute_all_silhouettes(
        self,
        X: np.ndarray,
        labels_dict: Dict[int, np.ndarray]
    ) -> Dict[int, float]:
        # Returns e.g. {3: 0.42, 4: 0.51, 5: 0.48}
        return {k: silhouette_score(X, labels_dict[k]) for k in labels_dict}

    def best_k(self, scores: Dict[int, float]) -> int:
        return max(scores, key=scores.get)
```

### Stability Bootstrapping

```python
    def stability_check(
        self,
        X: np.ndarray,
        k: int,
        n_trials: int    = 10,
        seeds: List[int] = None
    ) -> Dict[str, Any]:
        '''
        Runs KMeans with n_trials different random seeds.
        Measures variance in:
          1. Silhouette Score across trials
          2. Cluster size distribution (% deviation)
          3. Centroid position drift (Frobenius norm)
        Criterion: std(silhouettes) < 0.02 = STABLE
        '''
        scores, sizes = [], []
        for seed in seeds:
            km = KMeans(n_clusters=k, random_state=seed, n_init=20)
            km.fit(X)
            scores.append(silhouette_score(X, km.labels_))
            sizes.append(np.bincount(km.labels_) / len(X))
        return {
            'mean_silhouette': np.mean(scores),
            'std_silhouette':  np.std(scores),
            'stable':          np.std(scores) < 0.02,
            'size_cv':         np.std(sizes, axis=0).mean()
        }
```

---

## 2.6 Module: `persona_generator.py`

**Responsibility:** Analyzes cluster centroids and per-cluster aggregate statistics to assign human-readable business personas. Generates actionable recommendations per segment.

### Persona Classification Logic

```python
PERSONA_RULES = {
    'Champions':      {'recency': 'low',    'frequency': 'high',   'monetary': 'high'},
    'Loyal Customers':{'recency': 'low',    'frequency': 'high',   'monetary': 'medium'},
    'At-Risk':        {'recency': 'high',   'frequency': 'medium', 'monetary': 'medium'},
    'Bargain Hunters':{'recency': 'medium', 'frequency': 'low',    'monetary': 'low'},
    'Occasional':     {'recency': 'high',   'frequency': 'low',    'monetary': 'low'},
}

class PersonaGenerator:
    def compute_cluster_stats(self) -> pd.DataFrame
    def assign_personas(self, stats: pd.DataFrame) -> Dict[int, str]
    def generate_recommendations(self, persona: str) -> List[str]
    def export_personas(self, path: str) -> None
```

### Persona Output Schema

```json
{
  "0": {
    "persona": "Champions",
    "size": 1243,
    "pct_of_total": 0.18,
    "avg_recency": 12.4,
    "avg_frequency": 8.7,
    "avg_monetary": 2340.5,
    "recommendations": [
      "Offer exclusive loyalty rewards and early-access sales",
      "Request product reviews and referrals",
      "Enroll in VIP tier program"
    ]
  }
}
```

---

## 2.7 Module: `api_layer.py` (FastAPI Backend)

### Endpoint Specifications

| Method | Endpoint | Description | Response Type |
|--------|----------|-------------|---------------|
| GET | `/health` | Service liveness check | `JSON: {status: ok}` |
| GET | `/segment` | Return all cluster assignments + RFM features | JSON array of `CustomerSegment` |
| GET | `/personas` | Return all cluster personas + recommendations | JSON map of `PersonaCard` |
| GET | `/metrics` | Return silhouette scores, inertia, stability | `JSON: ModelMetrics` |
| POST | `/predict` | Predict segment for new customer RFM input | `JSON: {cluster_id, persona}` |

### Pydantic Models

```python
class RFMInput(BaseModel):
    recency:   float  # days since last purchase
    frequency: float  # number of transactions
    monetary:  float  # total spend

class CustomerSegment(BaseModel):
    customer_id: str
    cluster_id:  int
    persona:     str
    recency:     float
    frequency:   float
    monetary:    float

class ModelMetrics(BaseModel):
    best_k:           int
    silhouette_scores: Dict[int, float]
    inertia:          Dict[int, float]
    stability_report: Dict[str, Any]

@app.on_event('startup')
async def load_artifacts():
    app.state.model   = joblib.load('artifacts/kmeans_model.pkl')
    app.state.scaler  = joblib.load('artifacts/scaler.pkl')
    app.state.personas = json.load(open('artifacts/personas.json'))
```

### Logging Strategy

```python
logging.basicConfig(
    level=logging.INFO,
    format='{"time":"%(asctime)s","level":"%(levelname)s","msg":"%(message)s"}'
)
# Log every request: method, path, status_code, duration_ms
# Log pipeline steps: stage, rows_in, rows_out
# Log model events: k, silhouette_score, stability_status
```

---

## Error Handling Strategy

| Error Type | Module | Handling |
|------------|--------|----------|
| `FileNotFoundError` | `data_ingestion.py` | Log CRITICAL, raise with file path in message |
| `SchemaValidationError` | `data_ingestion.py` | Log ERROR, raise custom exception listing missing cols |
| `EmptyDataFrame` | `data_cleaning.py` | Log WARNING, raise ValueError with audit trail |
| `ConvergenceWarning` | `clustering_engine.py` | Catch, log WARNING, increase `max_iter` to 500 and retry |
| Low Silhouette (< 0.3) | `evaluation.py` | Log WARNING, flag in report, suggest re-running with different K |
| HTTP 422 | `api_layer.py` | FastAPI auto-validates Pydantic; return descriptive error body |
| Artifact Not Found | `api_layer.py` | Return HTTP 503 Service Unavailable with reload instruction |

---

## Success Criteria & Validation Checklist

| Criterion | Target | Validation Method |
|-----------|--------|-------------------|
| Silhouette Score | > 0.40 on best K | `ModelEvaluator.compute_silhouette()` |
| Model Stability | std(silhouette) < 0.02 across 10 seeds | `ModelEvaluator.stability_check()` |
| Data Audit Trail | Before/after counts at each cleaning step | `DataCleaningPipeline.get_audit_report()` |
| Business Personas | 3–5 interpretable, named cluster personas | `PersonaGenerator.assign_personas()` |
| API Latency | < 500ms p95 on `/segment` and `/personas` | FastAPI middleware timing + load test |
| Pipeline Runtime | < 5 minutes end-to-end on UCI dataset | Execution timing logs |
| Reproducibility | Identical output on same seed (42) | Automated test: run twice, assert labels array equality |
