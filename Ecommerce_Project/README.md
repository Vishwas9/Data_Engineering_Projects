# Retail Data Pipeline — Medallion Architecture on Databricks

An end-to-end ETL pipeline built on the [Online Retail II dataset](https://archive.ics.uci.edu/dataset/502/online+retail+ii) (~1M+ transactions, UK-based online retailer, 2009–2011), implementing **Medallion Architecture (Bronze → Silver → Gold)** using PySpark and Delta Lake on Databricks — the same core pattern used in production retail media pipelines.

## Why this project

I work daily with PySpark, Delta Lake, and Medallion Architecture in my current role, building pipelines for retail media data across 20+ global accounts. I wanted a public, self-contained version of the same patterns — one I could break, rebuild, and document properly outside the constraints of a live production system.

## Architecture

```
Raw Excel Files (2 sheets: 2009-2010, 2010-2011)
        │
        ▼
   BRONZE LAYER   — raw ingestion, combined + metadata tagged
        │
        ▼
   SILVER LAYER    — cleaned, standardized, split into
                     transactions vs. adjustments, DQ checked
        │
        ▼
   GOLD LAYER       — 4 analytics-ready tables
```

### Bronze — Raw Ingestion
- Ingested both sheets of the raw Excel source and combined them into a single dataset
- Cast mixed-type columns (Invoice, StockCode, Description) to consistent string types for Spark compatibility
- Tagged each record with `source_sheet`, `ingestion_timestamp`, and `_source_file` for traceability
- Wrote to Delta as `raw_retail_online`

### Silver — Cleaning & Standardization
- Standardized text fields (trimmed, uppercased) and removed duplicate records
- **Split real product transactions from non-product adjustment codes** (postage, bank charges, Amazon fees, manual adjustments) into two separate tables — `silver_transactions` and `silver_adjustments` — instead of treating all rows as sales
- Derived fields: `total_price` (quantity × price), `iscancelled` (invoice-prefix based), `guestcheckout` (null customer ID)
- Renamed columns to consistent snake_case for downstream use
- **Data quality check:** validated no unexpected negative-quantity rows outside of cancelled orders, and reconciled Silver row counts against Bronze

### Gold — Analytics-Ready Tables
| Table | Purpose |
|---|---|
| `gold_monthly_revenue_by_country` | Net revenue, order counts, cancellation rate, and AOV by month and country |
| `gold_top_products` | Products ranked by total revenue, quantity sold, and unique customers |
| `gold_cancellation_summary` | Cancellation rate and cancelled value by product, country, and month |
| `gold_customer_rfm` | Customer segmentation using **RFM analysis** (Recency, Frequency, Monetary) — quartile-scored and segmented into Champions, Loyal Customers, At Risk, and Lost |

## Tech Stack
- **PySpark** — distributed transformations across all layers
- **Delta Lake** — ACID transactions, schema enforcement, versioned storage
- **Databricks** — notebook execution and table management
- **Pandas** — initial Excel parsing prior to Spark DataFrame conversion

## Notes / Learnings
- Splitting adjustment codes from real transactions early (at Silver) kept downstream Gold aggregations accurate — mixing them would have inflated "sales" figures with postage and fees
- Data quality checks between layers (not just at the end) caught mismatches early, before they propagated into Gold tables

## Coming Next
- Orchestration with Airflow
- dbt-based transformation layer as an alternative to notebook-based Silver/Gold logic
