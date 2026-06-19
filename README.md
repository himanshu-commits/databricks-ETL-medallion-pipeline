
# Databricks Medallion ETL Pipeline with DAB & CI/CD

## Project Overview

This project demonstrates an end-to-end ETL pipeline built on Azure Databricks using the Medallion Architecture (Bronze, Silver, Gold). Data is incrementally ingested from a PostgreSQL (Neon) database using watermark-based processing and transformed through multiple layers for analytics and reporting.

The project also implements modern Data Engineering practices including:

* Incremental Data Loading
* Medallion Architecture
* Delta Tables
* Databricks Workflows
* Databricks Asset Bundles (DAB)
* GitHub Integration
* CI/CD using GitHub Actions

---

## Architecture

```text
PostgreSQL (Neon)
        │
        │ JDBC
        ▼
Incremental Load
        │
        ▼
Bronze Layer
(Historical Data)
        │
        ▼
Silver Layer
(Current State)
        │
        ▼
Gold Layer
(Business Aggregates)
        │
        ▼
Reporting / Analytics
```
<img width="809" height="373" alt="Screenshot 2026-06-19 at 5 17 37 PM" src="https://github.com/user-attachments/assets/0f149a3b-f47f-4d46-86f2-5eff2ee97b5e" />
---

## Medallion Architecture

### Bronze Layer

Purpose:

* Stores raw ingested data
* Preserves historical versions
* Append-only design
* Supports auditing and replay

Example:

```text
customer_id = 2, amount = 200
customer_id = 2, amount = 999
customer_id = 2, amount = 1200
```

---

### Silver Layer

Purpose:

* Cleans and standardizes data
* Maintains latest state
* Uses MERGE operations

Example:

```text
customer_id = 2, amount = 1200
```

Only the latest version is retained.

---

### Gold Layer

Purpose:

* Business-ready data
* Aggregations and reporting

Example:

```sql
SELECT
    region,
    SUM(amount) AS total_sales
FROM silver.sales
GROUP BY region
```

---

## Incremental Loading Strategy

The pipeline uses watermark-based incremental ingestion.

### Metadata Table

Stores the latest successfully processed timestamp.

```text
metadata.pipeline_metadata
```

### Workflow

1. Read watermark
2. Fetch records where:

```sql
updated_at > watermark
```

3. Append data to Bronze
4. Merge into Silver
5. Refresh Gold
6. Update watermark

Benefits:

* Efficient processing
* Reduced database load
* Scalable for large datasets

---

## Workflow Design

```text
02_incremental_load
          │
          ▼
03_bronze_to_silver
          │
          ▼
04_silver_to_gold
          │
          ▼
05_update_metadata
```

### 02_incremental_load

* Reads watermark
* Fetches changed records
* Appends to Bronze
* Creates staging table

### 03_bronze_to_silver

* Executes MERGE
* Updates existing records
* Inserts new records

### 04_silver_to_gold

* Builds business aggregates

### 05_update_metadata

* Updates watermark
* Enables future incremental loads

---

## Technologies Used

### Data Engineering

* Azure Databricks
* Delta Lake
* Databricks Workflows
* Databricks Asset Bundles (DAB)

### Database

* PostgreSQL (Neon)

### Development

* Python
* SQL
* Git
* GitHub

### CI/CD

* GitHub Actions
* Databricks CLI

---

## Databricks Asset Bundles (DAB)

DAB is used to manage Databricks resources as code.

### Benefits

* Workflow as Code
* Version Controlled Deployments
* Reproducible Environments
* Automated Resource Management

Project Structure:

```text
sales_medallion_pipeline/
│
├── databricks.yml
├── resources/
├── src/
├── tests/
└── README.md
```

---

## CI/CD Pipeline

GitHub Actions automatically validates and deploys bundle changes whenever code is pushed.

### CI/CD Flow

```text
VS Code
    │
    ▼
Git Commit
    │
    ▼
Git Push
    │
    ▼
GitHub Actions
    │
    ▼
Bundle Validate
    │
    ▼
Bundle Deploy
    │
    ▼
Databricks Updated
```

### GitHub Secrets

Required:

```text
DATABRICKS_HOST

DATABRICKS_TOKEN
```

---

## Deployment Strategy

### Code Lifecycle

```text
VS Code
    ↓
GitHub
    ↓
GitHub Actions
    ↓
DAB Deployment
    ↓
Databricks
```

### Data Lifecycle

```text
PostgreSQL
    ↓
Bronze
    ↓
Silver
    ↓
Gold
```

---

## Key Learning Outcomes

Through this project, the following concepts were implemented and understood:

* Medallion Architecture
* Incremental ETL Design
* Watermark-Based Processing
* Delta Lake Fundamentals
* Workflow Orchestration
* Databricks Asset Bundles
* Git Integration
* CI/CD Automation
* Production Data Engineering Practices

---

## Future Enhancements

Potential improvements include:

* Data Quality Validation
* Email Notifications
* Monitoring and Alerting
* MLflow Integration
* Vector Search Integration
* RAG-Based Analytics
* Model Serving
* Lakehouse Monitoring

---

## Author

Himanshu Gupta

Master's Student – Research in Computer and Systems Engineering

Technical University of Ilmenau

AI Engineer @Mercedes-Benz

If you have any doubts related to this project/architecture then contact me at himanshu.gupta@tu-ilmenau.de

