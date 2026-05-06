Marketing Campaign Lakehouse Pipeline
Overview

This project demonstrates an end-to-end Lakehouse pipeline built in Databricks using a Bronze → Silver → Gold architecture.

The pipeline processes marketing campaign CSV data, applies data quality validation, performs transformations, calculates business KPIs, and orchestrates the workflow using Databricks Jobs.

The goal of this project was to practice modern data engineering concepts such as:

Delta Lake
Incremental loading
Data quality validation
Lakehouse architecture
Pipeline orchestration
PySpark transformations
Bronze / Silver / Gold layering
Architecture
CSV Source Data
        ↓
Bronze Layer (Raw Delta Tables)
        ↓
Data Quality Checks
        ├── Invalid records → Error Audit Table
        └── Valid records continue
        ↓
Silver Layer (Cleaned / Validated Data)
        ↓
Gold Layer (Business Metrics & KPIs)
        ↓
Reporting / Analytics
Technologies Used
Python
PySpark
Databricks
Delta Lake
Databricks Jobs & Workflows
SQL
Lakehouse Architecture
Dataset

The dataset simulates marketing campaign performance data.

Example columns:

Column	Description
date	Campaign date
campaign_id	Campaign identifier
campaign_name	Campaign name
channel	Marketing channel
region	Region
impressions	Ad impressions
clicks	Ad clicks
conversions	Conversions
cost	Campaign cost
revenue	Revenue generated
last_updated	Incremental load timestamp
Project Structure
01_bronze_ingestion
02_silver_transform
03_gold_campaign_performance
04_data_quality_checks
Bronze Layer

The Bronze layer stores raw ingested data as Delta tables.

Features:

CSV ingestion
Delta table creation
Incremental MERGE logic
Raw data preservation

Example:

df.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("bronze_marketing_campaigns")
Incremental Loading

Incremental logic was implemented using Delta MERGE operations.

Example:

MERGE INTO bronze_marketing_campaigns AS target
USING incremental_campaign_updates AS source
ON target.campaign_id = source.campaign_id
AND target.date = source.date
AND target.region = source.region

WHEN MATCHED THEN
UPDATE SET *

WHEN NOT MATCHED THEN
INSERT *

This prevents full reloads and updates only changed/new records.

Data Quality Checks

A dedicated data quality notebook validates Bronze data before downstream processing.

Checks include:

NULL validation
Negative metric validation
Invalid business records

Invalid records are persisted into an audit/error table:

bronze_data_quality_errors

Example checks:

.filter(col("campaign_id").isNotNull())
.filter(col("impressions") > 0)
.filter(col("cost") > 0)
Silver Layer

The Silver layer stores cleaned and validated data.

Responsibilities:

Remove invalid records
Apply business validation rules
Create trusted analytical datasets

Example:

df_silver = df_bronze \
    .filter(col("campaign_id").isNotNull()) \
    .filter(col("impressions") > 0)
Gold Layer

The Gold layer enriches the Silver dataset with calculated business KPIs.

Calculated metrics:

CTR
Conversion Rate
ROI

Example:

.withColumn("ctr", round(col("clicks") / col("impressions"), 4))
.withColumn("conversion_rate", round(col("conversions") / col("clicks"), 4))
.withColumn("roi", round((col("revenue") - col("cost")) / col("cost"), 4))
Pipeline Orchestration

The pipeline is orchestrated using Databricks Jobs & Workflows.

Workflow:

bronze_ingestion
      ↓
data_quality_checks
      ↓
silver_transform
      ↓
gold_aggregation

This simulates a production-style orchestrated Lakehouse pipeline.

Key Concepts Practiced
Lakehouse architecture
Delta tables
ACID transactions
Incremental processing
Data quality monitoring
Pipeline orchestration
PySpark transformations
Business KPI calculations
Error auditing
Future Improvements

Potential future enhancements:

Partitioning strategy
Slowly Changing Dimensions (SCD)
Streaming ingestion
Unity Catalog integration
Automated alerting
CI/CD deployment
Power BI integration
Great Expectations / advanced data quality frameworks
Learning Outcomes

This project helped reinforce practical experience with:

Databricks notebooks
Delta Lake
PySpark
Data pipeline orchestration
Lakehouse design patterns
Data validation workflows
