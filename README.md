# Retail ETL Project

## 📋 Project Overview

This is a **Retail ETL (Extract, Transform, Load) Pipeline** built on Databricks using **Delta Lake** and the Medallion Architecture (Bronze → Silver → Gold) to process retail transaction data from Amazon S3. The pipeline is fully orchestrated for incremental loads and historical change tracking via SCD2 for dimensional modeling.

## 🎯 Business Purpose

The pipeline processes retail data including:
- **Customer information** (with change tracking)
- **Product catalog**
- **Store locations**
- **Sales transactions**

## 📁 Project Structure

```
Retail_ETL_Project/
├── 01_archival          # Archive old files from raw to archive folder
├── 02_bronze_load       # Load raw CSV files into Bronze layer
├── 03_silver_transform  # Clean and transform data into Silver layer
├── 04_gold_load         # Create dimensional model in Gold layer (one-time)
├── 05_scd2_merge        # Incremental SCD2 updates for all dimensions
├── 06_validation        # Data quality validation checks
```

## 🏗️ Architecture

### Data Layers

#### **Bronze Layer** (`capgeminipro.retail_bronze`)
- Raw data ingestion from S3
- Minimal transformations
- Data stored as-is from source
- Delta format for performance

**Tables:**
- `bronze_customers`
- `bronze_products`
- `bronze_stores`
- `bronze_sales`

#### **Silver Layer** (`capgeminipro.retail_silver`)
- Data cleaning and standardization
- Deduplication, null handling, cleansing
- Data quality checks
- Type validation

**Tables:**
- `silver_customers` - Cleaned customer data
- `silver_products_clean` - Valid products (price > 0)
- `silver_stores_clean` - Valid stores (non-null region)
- `silver_sales_final` - Valid sales transactions

**Rejected Data:**
- `rejected_products` - price = 0
- `rejected_stores` - NULL region
- `rejected_sales` - quantity <= 0

#### **Gold Layer** (`capgeminipro.retail_gold`)
- Business-ready dimensional model
- Implements SCD Type 2 for all dimensions
- Surrogate keys via identity columns
- Optimized for analytics/reporting

**Tables:**
- `dim_customers` - Customer dimension (SCD2)
- `dim_products` - Product dimension (SCD2)
- `dim_stores` - Store dimension (SCD2)
- `fact_sales` - Sales fact table (references SKs)

## 🔄 ETL Process Flow

### 1. File Archival (`01_archival`)
- Archive old files from raw/ to archive/
- Keep only latest in raw/

### 2. Bronze Load (`02_bronze_load`)
- Load CSVs from S3 using read_files()
- Create Delta tables in bronze

### 3. Silver Transformation (`03_silver_transform`)
- Trim, standardize, deduplicate, validate types
- Remove invalid products, stores, sales

### 4. Gold Load (`04_gold_load`)
- Create dimension/fact tables with SCD2 structure (StartDate, EndDate, IsCurrent='Y'/'N')
- One-time initial load

### 5. SCD2 Merge (`05_scd2_merge`)
- Incremental change tracking for all dimensions
- If attributes change → close old record (`IsCurrent='N'`, set EndDate), insert new version (`IsCurrent='Y'`)
- New entity → insert (`IsCurrent='Y'`)
- Fact table loads new transactions referencing latest SKs

### 6. Validation (`06_validation`)
- Data quality checks (nulls, duplicates, invalid prices/quantities, referential integrity)


## 📊 Data Sources

**S3 Bucket:** `s3://capgemini-retail-etl-shiva/`

- `raw/` - Latest files
- `archive/<entity>/` - Older files archived by entity
- `sftp/` - Legacy location

**File Naming:**
- `customers_src_DDMMYYYYHHMMSS.csv`
- `products_src_DDMMYYYYHHMMSS.csv`
- `stores_src_DDMMYYYYHHMMSS.csv`
- `sales_transactions_src_DDMMYYYYHHMMSS.csv`

## 🚀 How to Run

### Prerequisites
- Databricks workspace (Unity Catalog enabled)
- AWS S3 access
- Catalog: capgeminipro
- Cluster/serverless compute

### Execution Order
1. **01_archival** — archive files
2. **02_bronze_load** — load to bronze
3. **03_silver_transform** — cleanse
4. **04_gold_load** _(run ONCE)_ — initial dimension/fact creation
5. **05_scd2_merge** _(recurring)_ — incremental SCD2 loads
6. **06_validation** — quality checks
7. **07_logging** — runtime tracking

## 😎 Key Features

- **Medallion Architecture** — scalable, modular
- **Delta Lake** — ACID, time-travel
- **SCD Type 2** — historical tracking for dimensions
- **Surrogate Keys** — all dimensions use identity columns
- **Referential Integrity** — validated in silver/gold
- **Data Quality Checks** — bad data flagged
- **Process Logging** — end-to-end transparency
- **AWS + Databricks + Unity Catalog** — robust, governed

## 🛠️ Technologies

- Databricks (SQL, Python)
- Delta Lake
- Unity Catalog
- Apache Spark
- AWS S3

## 📝 Data Schemas

### Bronze Schema
- All columns loaded as strings

### Silver Schema
- **Customers:** CustomerID, CustomerName, Email, City, Address, LastUpdated (DATE)
- **Products:** ProductID, ProductName, Category, UnitPrice (DECIMAL)
- **Stores:** StoreID, StoreName, Region
- **Sales:** TransactionID, StoreID, ProductID, CustomerID, Quantity, TransactionDate (DATE)

### Gold Schema
- **dim_customers:** CustomerSK, CustomerID, CustomerName, Email, City, Address, StartDate, EndDate, IsCurrent
- **dim_products:** ProductSK, ProductID, ProductName, Category, UnitPrice, StartDate, EndDate, IsCurrent
- **dim_stores:** StoreSK, StoreID, StoreName, Region, StartDate, EndDate, IsCurrent
- **fact_sales:** SalesSK, TransactionID, CustomerSK, ProductSK, StoreSK, Quantity, TxnDate

## 🎯 Success Criteria
- Pipeline runs autonomously on Databricks
- Incremental loads are tracked in SCD2 dimensions (history preserved)
- Fact table references all valid surrogate keys
- Data is clean, validated, and ready for BI/reporting
- All steps are logged and validated for reliability

---
