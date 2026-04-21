# NYC Taxi Trip Analysis & Fare Prediction

> Analysing ~960 million taxi trips across New York City using Databricks, Spark SQL, and machine learning to surface business insights and predict trip fares.

---

## Table of Contents

- [Repository Structure](#repository-structure)
- [Project Overview](#project-overview)
- [Data Pipeline](#data-pipeline)
- [Business Questions](#business-questions)
- [Key Insights](#key-insights)
- [Machine Learning](#machine-learning)
- [Tech Stack](#tech-stack)
- [Results Summary](#results-summary)

---

## Repository Structure

| File | Description |
|------|-------------|
| `Download_Dataset.ipynb` | Part 1 — Data ingestion, cleaning, schema standardisation, location join, and final table creation in Databricks/PySpark |
| `Download_Dataset.html` | Static HTML export of the ingestion notebook (view without running) |
| `SQL_Business_Analysis.ipynb` | Part 2 — All business questions answered in Spark SQL, with outputs and commentary |
| `SQL_Business_Analysis.html` | Static HTML export of the SQL analysis notebook |
| `ML.ipynb` | Part 3 — Feature engineering, model training (Linear Regression + Random Forest), evaluation, and test-set predictions using sklearn |
| `ML.html` | Static HTML export of the ML notebook |
| `README.md` | This file |

---

## Project Overview

This project processes and analyses the full NYC TLC Yellow and Green taxi trip dataset (2014–2024) — approximately **1 billion raw records** — using a distributed Spark pipeline on Databricks. After cleaning and standardisation, roughly **960 million records** remain for analysis. Spark SQL drives the business analysis, and a scikit-learn ML pipeline predicts trip fare on a sampled subset.

---

## Data Pipeline

### Ingestion

- Source: NYC TLC Yellow and Green taxi datasets in **Parquet format**, 2014–2024
- Both datasets ingested independently before schema unification

### Cleaning & Validation

Records were removed if they met any of the following conditions:

| Rule | Description |
|------|-------------|
| Invalid timestamps | Dropoff before pickup, or dates outside 2014–2024 range |
| Unrealistic speed | Negative speed or exceeding NYC/highway limits |
| Unrealistic distance | Zero or implausibly long trip distances |
| Unrealistic duration | Trips too short (seconds) or too long (many hours) |
| Zero passengers | Trips with no recorded passengers |
| Duplicates | Exact duplicate records across key fields |

> Constraint: no more than 10% of raw records removed. Missing values in unused fields were not used as a removal criterion.

### Standardisation & Enrichment

- Unified schema across Yellow and Green datasets (column renaming, type casting)
- Joined with **TLC Location reference data** to enrich pickup/dropoff with borough and zone names

### Final Dataset

| Metric | Value |
|--------|-------|
| Raw records ingested | ~1 billion |
| Records after cleaning | ~960 million |
| Format | Delta table (Databricks) |
| Enrichment | Borough + zone for pickup & dropoff |

---

## Business Questions

### Q1 — Monthly trip summary (per year-month)
- Total trips, busiest day of week, busiest hour of day
- Average passengers per trip
- Average total amount paid per trip (USD)
- Average total amount paid per passenger (USD)

### Q2 — Trip statistics by taxi colour
- Average, median, min, max **trip duration** (minutes)
- Average, median, min, max **trip distance** (km)
- Average, median, min, max **speed** (km/h)

### Q3 — Granular revenue breakdown
Per colour × pickup/dropoff borough pair × month × day of week × hour:
- Total trips, average distance, average fare, total revenue

### Q4 — 2024 top revenue corridors
Top 10 pickup → dropoff borough pairs by total revenue, with each pair's share of 2024 total.

### Q5 — Tipping behaviour
- % of all trips where the driver received a tip
- Of tipped trips: % where tip was ≥ $3

### Q6 — Trip duration bins & driver optimisation
Trips classified into duration buckets (< 50 min / 50–100 / 100–200 / 200–300 / 300–600 / 600+ min).  
Per bin: average speed (km/h) and average distance per dollar (km/$).  
**Recommendation:** 30–60 minute trips offer the best balance of revenue per trip and trips per shift.

---

## Key Insights

| Finding | Detail |
|---------|--------|
| Peak demand | Evenings (~7 PM) and weekends |
| Average fare | $14–$16 |
| Average passengers | ~1.6 per trip |
| Top revenue corridor | Manhattan → Manhattan (~61% of total revenue) |
| Trips with tips | ~63% |
| Best duration window | 30–60 minutes |

---

## Machine Learning

### Objective
Predict `total_amount` for a trip. Features exclude `fare_amount` and `tolls_amount` (per spec).

### Data Split

| Split | Period |
|-------|--------|
| Train / validation | All trips before October 2024 |
| Test | October – December 2024 |

### Sample
**1% stratified sample** (~500,000 rows) used due to memory constraints with sklearn on a single node.

### Models

| Model | RMSE |
|-------|------|
| Baseline (mean predictor) | 16.25 |
| Linear Regression | 25.69 |
| **Random Forest** | **15.25** |

### Best Model: Random Forest

Random Forest beats the baseline (RMSE 15.25 vs 16.25) and significantly outperforms Linear Regression. Key reasons:

- Captures nonlinear relationships between distance, location, and fare
- Handles feature interactions (time × location × colour) without manual engineering
- More robust to fare outliers than regression

Linear Regression underperforms the mean baseline — raw categorical location features without encoding inflate variance beyond what a linear model can absorb.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Compute | Databricks (Apache Spark) |
| Query language | Spark SQL |
| ML framework | scikit-learn |
| Data format | Parquet / Delta table |
| Language | Python (PySpark + pandas) |

---

## Results Summary

| Metric | Value |
|--------|-------|
| Total trips analysed | ~960 million |
| Peak demand | Evenings (~7 PM), weekends |
| Top revenue corridor | Manhattan → Manhattan (~61%) |
| Average fare | $14–$16 |
| Trips with tips | ~63% |
| Best ML model | Random Forest |
| Best model RMSE | 15.25 |
| Baseline RMSE | 16.25 |

---

## Notes

- ML used a 1% sample due to single-node sklearn memory limits. A distributed SparkML pipeline would scale to the full dataset.
- Time-based train/test split is intentional — random splits leak future data into training, artificially inflating RMSE scores.
- Linear Regression's underperformance is expected without target-encoding of high-cardinality location features.
