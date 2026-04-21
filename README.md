Overview

This project analyses approximately 1 billion NYC taxi trips using Databricks, Spark SQL, and machine learning to generate business insights and predict trip fares.

Data Pipeline
Ingested Yellow and Green taxi datasets (Parquet)
Removed invalid records (incorrect timestamps, unrealistic distances/speeds, zero passengers, duplicates)
Standardized schemas across datasets
Merged datasets and joined with location reference data

Final dataset size: ~960 million records

Key Insights
Peak demand occurs in evenings (~7 PM) and on weekends
Average fare ranges between $14–$16 with ~1.6 passengers per trip
Manhattan to Manhattan trips contribute ~61% of total revenue
Around 63% of trips include tips, though large tips are rare
Trips between 30–60 minutes provide the best balance of revenue and volume
Machine Learning
Used 1% sample (~500,000 rows) due to memory constraints
Time-based split: training on data before Oct 2024, testing on Oct–Dec 2024

Model performance (RMSE):

Baseline: 16.25
Linear Regression: 25.69
Random Forest: 15.25

Best model: Random Forest, due to better generalization and ability to capture nonlinear patterns
