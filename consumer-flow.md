# Consumer Flow Diagram
## Retail Customer Segmentation

The following diagram illustrates the complete end-to-end interaction flow from Business User entry through the dashboard to final business output.

---

## Interaction Flow

```
╔══════════════════════════════════════════════════════════════════════════════╗
║           CONSUMER INTERACTION FLOW — RETAIL SEGMENTATION                  ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  ┌─────────────────────────────────────────────────────────────────────┐    ║
║  │                        BUSINESS USER                                │    ║
║  │                  Opens Dashboard / Web App                          │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║                                 ▼                                            ║
║  ┌─────────────────────────────────────────────────────────────────────┐    ║
║  │                 SCREEN 1: DASHBOARD LANDING PAGE                    │    ║
║  │  - Overview KPIs: Total Customers, Avg Spend, Avg Frequency         │    ║
║  │  - Date range selector (dataset period)                             │    ║
║  │  - Button: [Explore Segmentation]                                   │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │ User clicks: Explore Segmentation          ║
║                                 ▼                                            ║
║  ┌─────────────────────────────────────────────────────────────────────┐    ║
║  │            SCREEN 2: SELECT NUMBER OF CLUSTERS (K)                  │    ║
║  │  ┌──────────────────────────────────────────────────────────────┐   │    ║
║  │  │  [K=3]    [K=4 ✓ Recommended]    [K=5]                       │   │    ║
║  │  │  Silhouette: 0.39  |  0.51  |  0.47                          │   │    ║
║  │  │  Elbow Chart (Inertia vs K) displayed inline                  │   │    ║
║  │  └──────────────────────────────────────────────────────────────┘   │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║         ┌───────────────────────┼───────────────────────┐                   ║
║         ▼                       ▼                       ▼                   ║
║   [K=3 selected]         [K=4 selected]          [K=5 selected]             ║
║         └───────────────────────┼───────────────────────┘                   ║
║                                 │ (user confirms K)                          ║
║                                 ▼                                            ║
║  ┌─────────────────────────────────────────────────────────────────────┐    ║
║  │                  SCREEN 3: CLUSTER VISUALIZATION                    │    ║
║  │  ┌─────────────────────────┐  ┌──────────────────────────────────┐  │    ║
║  │  │  2D Scatter Plot        │  │  Cluster Distribution Bar Chart  │  │    ║
║  │  │  X-axis: Recency        │  │  Champions:      18%             │  │    ║
║  │  │  Y-axis: Monetary       │  │  Loyal:          32%             │  │    ║
║  │  │  Color: Cluster ID      │  │  At-Risk:        25%             │  │    ║
║  │  │  [Click point for info] │  │  Bargain Hunter: 25%             │  │    ║
║  │  └─────────────────────────┘  └──────────────────────────────────┘  │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │ User clicks on a cluster                   ║
║                                 ▼                                            ║
║  ┌─────────────────────────────────────────────────────────────────────┐    ║
║  │               SCREEN 4: CLUSTER DETAIL / METRICS VIEW               │    ║
║  │  ┌───────────────────────────────────────────────────────────────┐  │    ║
║  │  │  Cluster 0 — Champions                                        │  │    ║
║  │  │  Avg Recency: 12 days | Avg Freq: 8.7 | Avg Spend: £2340     │  │    ║
║  │  │  Silhouette Score: 0.51 | Stability: PASSED                   │  │    ║
║  │  │  Cluster Size: 1,243 customers (18% of base)                  │  │    ║
║  │  └───────────────────────────────────────────────────────────────┘  │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │ Decision: View Recommendations?            ║
║         ┌───────────────────────┴────────────────────┐                      ║
║         ▼                                             ▼                      ║
║  [YES — View Recs]                            [NO — Back to View]            ║
║         │                                                                    ║
║         ▼                                                                    ║
║  ┌─────────────────────────────────────────────────────────────────────┐    ║
║  │                SCREEN 5: BUSINESS RECOMMENDATIONS                   │    ║
║  │  ┌───────────────────────────────────────────────────────────────┐  │    ║
║  │  │  ✔ Offer exclusive loyalty rewards and early-access sales     │  │    ║
║  │  │  ✔ Request product reviews and referrals                      │  │    ║
║  │  │  ✔ Enroll in VIP tier program                                 │  │    ║
║  │  └───────────────────────────────────────────────────────────────┘  │    ║
║  │  [Export to PDF]    [Send to CRM]    [Download Customer List]        │    ║
║  └─────────────────────────────────────────────────────────────────────┘    ║
║                                                                              ║
║  DECISION POINTS:                                                            ║
║  ◆ K Selection      → User chooses or system recommends optimal K           ║
║  ◆ Cluster Drill-Down → User clicks any cluster on scatter/bar chart        ║
║  ◆ Action Export    → Export customer list for CRM integration              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## Screen-by-Screen Summary

| Screen | Name | User Action |
|--------|------|-------------|
| Screen 1 | Dashboard Landing Page | Click "Explore Segmentation" |
| Screen 2 | Select Number of Clusters (K) | Choose K=3, K=4, or K=5; confirm |
| Screen 3 | Cluster Visualization | Click on a cluster point or bar |
| Screen 4 | Cluster Detail / Metrics View | Decide whether to view recommendations |
| Screen 5 | Business Recommendations | Export to PDF, send to CRM, or download list |

## Decision Points

- **K Selection** — User chooses or system recommends optimal K (default: K=4, Silhouette = 0.51)
- **Cluster Drill-Down** — User clicks any cluster on the scatter or bar chart to view details
- **Action Export** — User exports the customer list to CRM or downloads as PDF
