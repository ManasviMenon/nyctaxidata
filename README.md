#  NYC Taxi Trip Analysis & Fare Prediction

> Analysing ~960 million taxi trips across New York City using Databricks, Spark SQL, and machine learning to surface business insights and predict trip fares.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Data Pipeline](#data-pipeline)
- [Dataset](#dataset)
- [Key Business Insights](#key-business-insights)
- [Machine Learning](#machine-learning)
- [Tech Stack](#tech-stack)
- [Results Summary](#results-summary)

---

## Project Overview

This project processes and analyses the full NYC Yellow and Green taxi trip dataset — approximately **1 billion raw records** — using a distributed Spark pipeline on Databricks. After cleaning and standardisation, roughly **960 million records** remain for analysis. A machine learning pipeline then trains and evaluates fare prediction models on a stratified sample.

---

## Data Pipeline

### Ingestion

- Source: NYC TLC Yellow and Green taxi datasets in **Parquet format**
- Both datasets ingested independently before schema unification

### Cleaning & Validation

Records were removed if they met any of the following conditions:

| Rule | Description |
|------|-------------|
| Invalid timestamps | Trip start/end outside plausible date ranges |
| Unrealistic distances | Zero-distance or physically impossible trip lengths |
| Unrealistic speeds | Average speed exceeding plausible vehicle limits |
| Zero passengers | Trips with no recorded passengers |
| Duplicates | Exact duplicate records across key fields |

### Standardisation

- Unified schema across Yellow and Green datasets (column renaming, type casting)
- Joined with **TLC Location reference data** to enrich pickup/dropoff zones

### Final Dataset

| Metric | Value |
|--------|-------|
| Raw records ingested | ~1 billion |
| Records after cleaning | ~960 million |
| Format | Parquet (partitioned) |
| Enrichment | Zone-level location data (pickup & dropoff) |

---

## Key Business Insights

### 1. Peak Demand
Evening hours (~7 PM) see the highest trip volumes. Weekend demand is elevated relative to weekday baselines, suggesting leisure and hospitality as primary drivers outside business hours.

### 2. Fare & Occupancy Averages
- Average fare: **$14–$16**
- Average passengers per trip: **~1.6**

Most trips are short, solo or paired, and relatively low-value individually — volume is the revenue driver.

### 3. Revenue Concentration
**Manhattan → Manhattan** trips account for approximately **61% of total revenue**, confirming the borough as the economic core of the network.

### 4. Tipping Behaviour
- ~**63% of trips** include a tip
- Large tips are rare; the distribution is heavily right-skewed
- Tip presence is a majority behaviour, but tip magnitude is not a reliable revenue lever

### 5. High-Value Trip Duration
Trips in the **30–60 minute** range offer the best balance of:
- Per-trip revenue (longer than short hops)
- Trip volume (short enough to complete multiple per shift)

This window represents the sweet spot for driver earnings optimisation.

---

## Machine Learning

### Objective
Predict trip fare amount given features available at trip start (pickup location, time, distance, passenger count, etc.).

### Data Split Strategy
A **time-based split** was used to simulate real-world deployment conditions:

| Split | Period |
|-------|--------|
| Training set | All trips before October 2024 |
| Test set | October – December 2024 |

### Sample
Due to memory constraints, a **1% stratified sample** (~500,000 rows) was used for modelling. Stratification preserved temporal and geographic distributions.

### Models Evaluated

| Model | RMSE |
|-------|------|
| Baseline (mean predictor) | 16.25 |
| Linear Regression | 25.69 |
| **Random Forest** | **15.25** |

### Best Model: Random Forest

Random Forest outperformed both the baseline and Linear Regression. Key reasons:

- **Nonlinear relationships**: Fare is influenced by zone-level pricing, surge patterns, and distance in ways that are not well-captured by linear models.
- **Feature interactions**: The model can capture interactions between time-of-day, location, and distance without explicit feature engineering.
- **Robustness to outliers**: Tree-based methods are less sensitive to fare outliers than regression.

Linear Regression underperformed the baseline (RMSE 25.69 vs 16.25), indicating that the raw features without polynomial or interaction terms produce worse-than-average predictions — likely due to high variance in trip types that linearity cannot resolve.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Compute | Databricks (Apache Spark) |
| Query language | Spark SQL |
| ML framework | MLlib (PySpark) |
| Data format | Parquet |
| Orchestration | Databricks Jobs / Notebooks |

---

## Results Summary

| Metric | Value |
|--------|-------|
| Total trips analysed | ~960 million |
| Peak demand window | Evenings (~7 PM), weekends |
| Top revenue corridor | Manhattan → Manhattan (~61%) |
| Average fare | $14–$16 |
| Trips with tips | ~63% |
| Best ML model | Random Forest |
| Best model RMSE | 15.25 |
| Baseline RMSE | 16.25 |

---

## Notes

- ML experiments used a 1% sample due to Spark driver memory limits. Full-dataset training via Spark's distributed `RandomForestRegressor` is feasible with appropriate cluster sizing.
- The time-based train/test split is intentional — random splits would leak future information into training, inflating performance metrics.
- Linear Regression's underperformance relative to the mean baseline is a known pattern when raw categorical location features (zone IDs) are not properly encoded; further feature engineering (target encoding, distance bins) would likely close the gap.
