# E-Commerce Product Analysis — November 2019

An end-to-end product, session, retention, cart abandonment, and time-to-purchase analysis of the [eCommerce behavior dataset](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store) using **DuckDB** for low-RAM querying, **Matplotlib** for visualization, and **Jupyter Notebook** in VS Code.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        LOCAL MACHINE                                │
│                                                                     │
│   ┌──────────────┐     ┌──────────────────────────────────────┐     │
│   │              │     │        VS Code + Jupyter              │    │
│   │  2019-Nov    │     │                                       │    │
│   │  .csv        │◄────┤   analysis.ipynb                      │    │
│   │  (~9 GB)     │     │   ┌───────────────────────────────┐   │    │
│   │  stays on    │     │   │  Cell 1 — Imports + Config    │   │    │
│   │  disk        │     │   │  Cell 2 — DuckDB Connect      │   │    │
│   └──────┬───────┘     │   │  Cell 3 — Dataset Overview    │   │    │
│          │             │   │  Cell 4 — Missing Values      │   │    │
│          │  lazy scan  │   │  Cell 5 — Event Distribution  │   │    │
│          ▼             │   │  Cell 6 — Conversion Funnel   │   │    │
│   ┌──────────────┐     │   │  ...                          │   │    │
│   │              │     │   │  Cell N — Business Insights   │   │    │
│   │   DuckDB     │     │   └───────────────────────────────┘   │    │
│   │   Engine     │     │                                       │    │
│   │              │     │   inline charts (plt.show())          │    │
│   │ • columnar   │     └──────────────────┬────────────────────┘    │
│   │ • lazy scan  │                        │                         │
│   │ • < 1GB RAM  │                        │ saves                   │
│   └──────┬───────┘                        ▼                         │
│          │                  ┌─────────────────────────┐             │
│          │  small DataFrames│  product_analysis_output/│            │
│          └─────────────────►│  ├── charts (01–23 PNG) │             │
│                             │  ├── analysis_summary   │             │
│                             │  │   .xlsx              │             │
│                             │  ├── session_retention  │             │
│                             │  │   _analysis.xlsx     │             │
│                             │  └── cart_time_insights │             │
│                             │      .xlsx              │             │
│                             └─────────────────────────┘             │
└─────────────────────────────────────────────────────────────────────┘
```

### How DuckDB keeps RAM under control

```
Without DuckDB (pandas):
  CSV (9 GB) ──► pd.read_csv() ──► RAM: 18–27 GB  ──► CRASH

With DuckDB:
  CSV (9 GB) ──► DuckDB scan ──► only needed columns/rows ──► small DataFrame ──► RAM: < 1 GB
```

---

## Dataset

| Property | Value |
|---|---|
| Source | Kaggle — eCommerce behavior data from multi-category store |
| File | `2019-Nov.csv` |
| Size | ~9 GB |
| Rows | ~67 million |
| Period | November 2019 |

### Columns

| Column | Type | Notes |
|---|---|---|
| `event_time` | timestamp | When the event occurred |
| `event_type` | string | `view`, `cart`, or `purchase` |
| `product_id` | int | Unique product identifier |
| `category_id` | int | Unique category identifier |
| `category_code` | string | Human-readable category path — **has missing values** |
| `brand` | string | Product brand — **has missing values** |
| `price` | float | Product price in USD |
| `user_id` | int | Unique user identifier |
| `user_session` | string | Groups all events belonging to one visit |

---

## Project Structure

```
project/
│
├── E-commerce.ipynb                     # Single notebook — all analysis
├── 2019-Nov.csv                       # Raw dataset (download from Kaggle)
├── README.md                          # This file
│
└── product_analysis_output/           # Auto-created on first run
    │
    ├── ── Product Analysis ──
    ├── 01_event_type_distribution.png
    ├── 02_conversion_funnel.png
    ├── 03_top_categories.png
    ├── 04_top_brands.png
    ├── 05_purchase_price_dist.png
    ├── 06_avg_price_by_category.png
    ├── 07_top_products.png
    ├── 08_revenue_by_category.png
    ├── 09_daily_trends.png
    ├── 10_hourly_purchase_pattern.png
    ├── 11_category_conversion_rates.png
    ├── 12_brand_conversion_rates.png
    │
    ├── ── Session & Retention ──
    ├── 13_session_distributions.png
    ├── 14_session_depth_funnel.png
    ├── 15_conversion_per_session_category.png
    ├── 16_user_retention_segments.png
    ├── 17_retention_cohort_heatmap.png
    │
    ├── ── Cart & Time-to-Purchase ──
    ├── 18_cart_abandonment_funnel.png
    ├── 19_abandonment_by_category.png
    ├── 20_abandonment_by_price.png
    ├── 21_time_to_purchase.png
    ├── 22_ttp_by_category.png
    ├── 23_dow_purchase_pattern.png
    │
    ├── ── Excel Exports ──
    ├── analysis_summary.xlsx
    ├── session_retention_analysis.xlsx
    └── cart_time_insights.xlsx
```

---

## Notebook Structure

The single `analysis.ipynb` is organised into 4 parts. Each part maps to a comment block in the code — paste each block into its own cell and run with `Shift+Enter`.

### Part 1 — Setup

| Cell | Content |
|---|---|
| 1 | Imports (`duckdb`, `pandas`, `matplotlib`, etc.) |
| 2 | Config — set `FILE_PATH`, `OUTPUT_DIR`, `CHUNKSIZE` |
| 3 | DuckDB connect + `q()` helper function |

### Part 2 — Product Analysis (Sections 1–14)

| Section | Chart | What it shows |
|---|---|---|
| 1 — Dataset Overview | — | Shape, date range, unique users/products |
| 2 — Missing Values | — | % missing in `category_code` and `brand` |
| 3 — Event Distribution | `01` | Count of views, carts, purchases |
| 4 — Conversion Funnel | `02` | View → cart → purchase drop-off |
| 5 — Top Categories | `03` | Top 10 by views and purchases |
| 6 — Top Brands | `04` | Top 15 by views and purchases |
| 7 — Price Analysis | `05` `06` | Distribution + avg by category |
| 8 — Top Products | `07` | Most viewed and most purchased |
| 9 — Revenue | `08` | Total revenue, AOV, revenue by category |
| 10 — Daily Trends | `09` | All event types across November |
| 11 — Hourly Pattern | `10` | Peak purchase hours |
| 12 — Category Conv. | `11` | View→purchase rate by category |
| 13 — Brand Conv. | `12` | View→purchase rate by brand |

### Part 3 — Session & Retention Analysis

| Section | Chart | What it shows |
|---|---|---|
| S1 — Session Metrics | `13` | Avg length, events/session, conv/session |
| S2 — Session Depth | `14` | How deep users go per session |
| S3 — Conv per Session | `15` | Best-converting categories at session level |
| S4 — User Retention | `16` | New vs returning vs power users |
| S5 — Cohort Heatmap | `17` | 7-day retention heatmap by cohort day |

### Part 4 — Cart Abandonment + Time-to-Purchase + Insights

| Section | Chart | What it shows |
|---|---|---|
| C1 — Abandonment Overview | `18` | Overall abandonment rate + donut chart |
| C2 — Abandonment by Category | `19` | Which categories lose the most carts |
| C3 — Abandonment by Price | `20` | Whether price range affects abandonment |
| T1 — TTP Distribution | `21` | Time from first view to purchase |
| T2 — TTP by Category | `22` | Which categories take longest to convert |
| T3 — Day of Week | `23` | Best days for purchases and revenue |
| FINAL — Business Insights | — | Auto-generated findings + recommendations |

---

## Key Metrics

| Metric | Description |
|---|---|
| View → Purchase rate | % of views that eventually result in a purchase |
| Cart abandonment rate | % of cart sessions that never converted |
| Avg session length | Mean duration (minutes) per user session |
| Events per session | Avg number of actions per session |
| Conversion per session | % of sessions containing at least one purchase |
| Median time-to-purchase | Median minutes from first product view to buying |
| 7-day retention | % of Day-0 users still active on Day 1–6 |
| Recoverable revenue | Estimated revenue if abandoned carts were partially recovered |

---

## Business Insights (Auto-generated)

The final notebook cell automatically generates dynamic insights from the computed metrics:

| # | Topic | Example finding |
|---|---|---|
| 1 | Cart Abandonment | Abandonment rate, worst category, estimated recoverable revenue |
| 2 | Conversion Gap | Where the biggest drop-off actually happens |
| 3 | Time-to-Purchase | Impulse vs considered buyer behaviour |
| 4 | Purchase Timing | Peak hour + best day for ad scheduling |
| 5 | Session Quality | Session depth and bounce reduction strategies |
| 6 | Data Quality | Impact of missing brand/category on conversion |

---

## Excel Outputs

| File | Sheets |
|---|---|
| `analysis_summary.xlsx` | Event Summary, Top Products, Top Brands, Daily Trends, Category Conv, Brand Conv, Revenue |
| `session_retention_analysis.xlsx` | Session Metrics, Session Depth Funnel, Conv Per Session, User Retention, Cohort Retention, Top Sessions, User Segments |
| `cart_time_insights.xlsx` | Cart Abandonment, Abandonment by Category, Abandonment by Price, TTP Stats, TTP Distribution, TTP by Category, Day of Week, Business Insights |

---

## Installation

```bash
pip install duckdb pandas matplotlib seaborn openpyxl ipykernel
```

Open VS Code → install the **Jupyter** extension → open `analysis.ipynb` → Run All.

> Update `FILE_PATH` in the config cell to point to your `2019-Nov.csv` before running.

---

## Missing Value Handling

Both `category_code` and `brand` have significant missing values. All queries use SQL `COALESCE` to substitute `'unknown_category'` and `'unknown_brand'` inline — preserving full row counts while excluding unknowns from brand/category charts.

---

## Notes

- All timestamps are **UTC**. Hourly and daily patterns reflect UTC, not local store time.
- Revenue is computed from the `price` column on `purchase` events only.
- If running on a sample instead of the full 67M row file, revenue totals and absolute counts will differ from the true November figures. Ratios and rates (conversion %, abandonment %) remain statistically valid.
- The last 6 days of November show incomplete cohort data in the retention heatmap — this is expected behaviour, not a bug.

---

## Requirements

| Package | Min version | Purpose |
|---|---|---|
| `duckdb` | 0.9 | Query CSV without loading into RAM |
| `pandas` | 1.5 | DataFrame operations on query results |
| `matplotlib` | 3.5 | All charts and visualizations |
| `seaborn` | 0.12 | Chart styling |
| `openpyxl` | 3.0 | Excel export |
| `ipykernel` | 6.0 | Jupyter notebook support in VS Code |
