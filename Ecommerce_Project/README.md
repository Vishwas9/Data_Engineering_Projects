# Retail E-Commerce Medallion Pipeline

An end-to-end data pipeline built on Databricks that processes raw retail e-commerce data through a Bronze → Silver → Gold Medallion Architecture, producing clean, analysis-ready datasets for downstream reporting and analytics.

## Problem statement
Raw retail transaction data arrives from [source — e.g. POS systems / CSV drops / API feeds] in an inconsistent, unvalidated state — missing fields, schema drift across batches, and duplicate records. This pipeline ingests that raw data and progressively refines it into trustworthy, query-ready tables that a BI team or analyst can rely on without doing their own cleanup.

## Architecture

```
Raw Source Files
      │
      ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   BRONZE    │────▶│   SILVER    │────▶│    GOLD     │
│ Raw ingest  │     │  Cleaned &  │     │  Aggregated │
│ (as-is)     │     │  validated  │     │  business   │
│             │     │             │     │  tables     │
└─────────────┘     └─────────────┘     └─────────────┘
   Delta Lake          Delta Lake          Delta Lake
   (Volumes)          (DQ checks,         (star schema /
                      schema evolution)    reporting-ready)
```

**Bronze** — Raw data ingested as-is from source, using Databricks Volumes for structured file landing. No transformation; preserves full fidelity and history of source data.

**Silver** — Cleaned and validated data. Applies schema evolution handling so the pipeline doesn't break when source schemas shift, and enforces data quality rules (null checks, type validation, dedup) — records failing these checks are quarantined rather than silently dropped.

**Gold** — Business-level aggregated tables (e.g. daily sales by category, customer lifetime value, inventory turnover) — modeled for direct consumption by BI tools/analysts.

## Tech stack
- **Processing:** PySpark, Apache Spark
- **Storage/Format:** Delta Lake
- **Platform:** Databricks (Volumes, Delta Live Tables)
- **Language:** SQL, Python

## Project structure
```
├── bronze/
│   └── bronze_ingstion.py
├── silver/
│   ├── silver_transformations.py
├── gold/
│   └── gold_tables.py
└── README.md
```

## Sample data / scale
- Records processed: [e.g. ~2M synthetic retail transactions]
- Source format: [XLSX]

## Future improvements
- Add streaming ingestion (Structured Streaming / Kafka) for near-real-time updates
- Introduce dbt for the Gold-layer transformation logic
- Add data contracts between Bronze and Silver

## Author
Vishwas G — [LinkedIn](https://www.linkedin.com/in/vishwas-gangaraju-1169901b3/) · [Email](mailto:vishwasgangaraju09@gmail.com)
