# E-Commerce Product Analysis — November 2019

A end-to-end product, session, retention, cart abandonment, and time-to-purchase analysis of the [eCommerce behavior dataset](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store) using **DuckDB** (for low-RAM querying) and **Matplotlib** (for visualization).

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
| `category_code` | string | Human-readable category path (has missing values) |
| `brand` | string | Product brand (has missing values) |
| `price` | float | Product price in USD |
| `user_id` | int | Unique user identifier |
| `user_session` | string | Session identifier — groups events by visit |

---

## Why DuckDB instead of pandas?

Loading 67M rows into pandas requires 18–27 GB of RAM, which crashes most laptops. DuckDB queries the CSV directly from disk using columnar scanning — each query uses under 1 GB of RAM regardless of file size, and returns a small result DataFrame to matplotlib.

```
CSV on disk (9 GB)  →  DuckDB engine (lazy scan)  →  Small DataFrame  →  Chart
```

---

## Project Structure

```
project/
│
├── ecommerce_duckdb_analysis.py (Main analysis (sections 1–14))
├──session_retention_analysis (Session + retention analysis)
├── cart_time_insights_analysis (Cart abandonment + TTP + insights)
│
├── 2019-Nov.csv                      # Raw dataset (download from Kaggle)
│
└── product_analysis_output/          # Auto-created on first run
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
    ├── 13_session_distributions.png
    ├── 14_session_depth_funnel.png
    ├── 15_conversion_per_session_category.png
    ├── 16_user_retention_segments.png
    ├── 17_retention_cohort_heatmap.png
    ├── 18_cart_abandonment_funnel.png
    ├── 19_abandonment_by_category.png
    ├── 20_abandonment_by_price.png
    ├── 21_time_to_purchase.png
    ├── 22_ttp_by_category.png
    ├── 23_dow_purchase_pattern.png
    ├── analysis_summary.xlsx
    ├── session_retention_analysis.xlsx
    └── cart_time_insights.xlsx
```

---

## Installation

```bash
pip install duckdb pandas matplotlib seaborn openpyxl ipykernel
```

Recommended: run in **VS Code** with the Jupyter extension installed.

---

## Usage

### Option A — Merge into one file and run once

**Mac / Linux**
```bash
cat ecommerce_duckdb_analysis.py \
    session_retention_analysis.py \
    cart_time_insights_analysis.py > full_analysis.py

python full_analysis.py
```

**Windows**
```cmd
type ecommerce_duckdb_analysis.py session_retention_analysis.py cart_time_insights_analysis.py > full_analysis.py
python full_analysis.py
```

### Option B — Run cell by cell in Jupyter (VS Code)

Open `full_analysis.py` as a notebook in VS Code. Each analysis section is separated by a comment block — paste each section into its own cell and run with `Shift+Enter`.

> Before running, update `FILE_PATH` at the top of `ecommerce_duckdb_analysis.py` to your actual CSV path.

---

## Analysis Sections

### Part 1 — Product Analysis (`ecommerce_duckdb_analysis.py`)

| # | Section | What it shows |
|---|---|---|
| 1 | Dataset Overview | Shape, date range, unique users/products |
| 2 | Missing Value Analysis | % missing in `category_code` and `brand` |
| 3 | Event Type Distribution | Count of views, carts, purchases |
| 4 | Conversion Funnel | View → cart → purchase drop-off rates |
| 5 | Top Categories | Top 10 by views and by purchases |
| 6 | Top Brands | Top 15 by views and by purchases |
| 7 | Price Analysis | Distribution, avg by category, stats by event type |
| 8 | Top Products | Most viewed and most purchased products with metadata |
| 9 | Revenue Analysis | Total revenue, avg order value, revenue by category |
| 10 | Daily Trends | All event types plotted across November |
| 11 | Hourly Purchase Pattern | Which hours of the day drive the most purchases |
| 12 | Category Conversion Rates | View → purchase rate per category |
| 13 | Brand Conversion Rates | View → purchase rate per brand |

### Part 2 — Session & Retention Analysis (`session_retention_analysis`)

| # | Section | What it shows |
|---|---|---|
| S1 | Session Metrics | Avg session length, events per session, conversion per session |
| S2 | Session Depth Funnel | How many sessions reached view / cart / purchase |
| S3 | Conversion per Session by Category | Which categories convert best at session level |
| S4 | User Retention Segments | New vs returning users, power users |
| S5 | 7-Day Retention Cohort Heatmap | % of users returning on Day 1–6 after first visit |

### Part 3 — Cart Abandonment + Time-to-Purchase (`cart_time_insights_analysis`)

| # | Section | What it shows |
|---|---|---|
| C1 | Cart Abandonment Overview | Overall abandonment rate, estimated lost revenue |
| C2 | Abandonment by Category | Which categories lose the most carts |
| C3 | Abandonment by Price Range | Whether price affects abandonment rate |
| T1 | Time-to-Purchase Distribution | How long from first view to purchase |
| T2 | TTP by Category | Which categories take longest to convert |
| T3 | Day of Week Patterns | Best days for purchases and revenue |

---

## Key Metrics Produced

| Metric | Description |
|---|---|
| View → Purchase rate | % of views that eventually result in a purchase |
| Cart abandonment rate | % of cart sessions that never converted |
| Avg session length | Mean time (minutes) per user session |
| Events per session | Avg number of actions per session |
| Conversion per session | % of sessions that included a purchase |
| Median time-to-purchase | Median minutes from first product view to buying it |
| 7-day retention | % of Day-0 users still active on Day 1, 2 ... 6 |
| Recoverable revenue | Estimated revenue if abandoned carts were partially recovered |

---

## Excel Outputs

| File | Sheets |
|---|---|
| `analysis_summary.xlsx` | Event Summary, Top Products, Top Brands, Daily Trends, Category Conv, Brand Conv, Revenue |
| `session_retention_analysis.xlsx` | Session Metrics, Session Depth Funnel, Conv Per Session, User Retention, Cohort Retention, Top Sessions, User Segments |
| `cart_time_insights.xlsx` | Cart Abandonment, Abandonment by Category, Abandonment by Price, TTP Stats, TTP Distribution, TTP by Category, Day of Week, Business Insights |

---

## Business Insights (Auto-generated)

The final section of `cart_time_insights_analysis` automatically generates dynamic insights from the analysis results, covering:

1. **Cart Abandonment** — abandonment rate, worst category, estimated recoverable revenue
2. **Conversion Gap** — where the biggest drop-off actually occurs (cart→purchase vs view→cart)
3. **Time-to-Purchase** — impulse vs considered buyer behaviour, retargeting opportunities
4. **Purchase Timing** — peak hour and best day of week for ad scheduling
5. **Session Quality** — session depth recommendations, bounce reduction strategies
6. **Data Quality** — impact of missing brand/category on conversion

---

## Missing Value Handling

Both `category_code` and `brand` have significant missing values. Rather than dropping rows, all queries use SQL `COALESCE` to substitute `'unknown_category'` and `'unknown_brand'` inline, preserving full row counts while excluding unknowns from brand/category-specific charts.

---

## Notes

- All timestamps in the dataset are **UTC**. Hourly and daily patterns reflect UTC time, not local store time.
- Revenue figures are based on the `price` column from purchase events only.
- Session conversion rate counts any session containing at least one purchase event.
- The 7-day cohort heatmap is most meaningful when the dataset covers a full month — the last 6 days of November will show incomplete cohort data by design.
- If running on the 500K sample instead of the full 67M row file, revenue totals and absolute counts will not reflect the true November figures. Ratios and rates (conversion %, abandonment %) remain valid.

---

## Requirements

| Package | Version | Purpose |
|---|---|---|
| `duckdb` | >= 0.9 | Query CSV without loading into RAM |
| `pandas` | >= 1.5 | DataFrame operations on query results |
| `matplotlib` | >= 3.5 | All charts and visualizations |
| `seaborn` | >= 0.12 | Chart styling |
| `openpyxl` | >= 3.0 | Excel export |
| `ipykernel` | >= 6.0 | Jupyter notebook support in VS Code |

```bash
pip install duckdb pandas matplotlib seaborn openpyxl ipykernel
```
