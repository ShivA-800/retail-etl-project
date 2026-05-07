# Retail ETL Project

## 📋 Project Overview

This is a **Retail ETL (Extract, Transform, Load) Pipeline** built on Databricks using **Delta Lake** architecture. The project follows the **Medallion Architecture** (Bronze → Silver → Gold) to process retail transaction data from S3 storage.

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
├── 04_gold_load         # Create dimensional model in Gold layer
├── 05_scd2_merge        # Implement SCD Type 2 for customer dimension
├── 06_validation        # Data quality validation checks
└── 07_logging           # Process logging and monitoring
```

## 🏗️ Architecture

### Data Layers

#### **Bronze Layer** (`capgeminipro.retail_bronze`)
- **Raw data ingestion** from S3
- Minimal transformations
- Data stored as-is from source

**Tables:**
- `bronze_customers`
- `bronze_products`
- `bronze_stores`
- `bronze_sales`

#### **Silver Layer** (`capgeminipro.retail_silver`)
- **Data cleaning and standardization**
- Remove duplicates
- Handle null values
- Data type conversions
- Data quality checks

**Tables:**
- `silver_customers` - Cleaned customer data
- `silver_products_clean` - Valid products (price > 0)
- `silver_stores_clean` - Valid stores (with regions)
- `silver_sales_final` - Valid sales transactions

**Rejected Data Tables:**
- `rejected_products` - Products with price = 0
- `rejected_stores` - Stores with NULL regions
- `rejected_sales` - Sales with quantity <= 0

#### **Gold Layer** (`capgeminipro.retail_gold`)
- **Business-ready dimensional model**
- Implements Slowly Changing Dimensions (SCD Type 2)
- Optimized for analytics and reporting

**Tables:**
- `dim_customers` - Customer dimension with SCD Type 2
- `dim_products` - Product dimension
- `dim_stores` - Store dimension
- `fact_sales` - Sales fact table

## 🔄 ETL Process Flow

### Step 1: File Archival (01_archival)
```python
# Archives old customer files from raw/ to archive/ folder
# Keeps only the latest file in raw/ folder
```

### Step 2: Bronze Load (02_bronze_load)
```sql
# Loads CSV files from S3 using read_files()
# Creates Delta tables in bronze schema
```

### Step 3: Silver Transformation (03_silver_transform)
**Transformations Applied:**
- **Customers:** Trim spaces, standardize names (INITCAP), lowercase emails, parse dates
- **Products:** Remove products with price = 0
- **Stores:** Remove stores without regions
- **Sales:** Remove transactions with quantity <= 0

### Step 4: Gold Load (04_gold_load)
**Dimensional Model:**
- Creates dimension tables with surrogate keys
- Implements identity columns for auto-incrementing keys

### Step 5: SCD Type 2 Implementation (05_scd2_merge)
**Customer Dimension Change Tracking:**
- Tracks historical changes in customer attributes
- Fields tracked: `CustomerName`, `City`, `Address`
- Uses `IsCurrent` flag (Y/N)
- Maintains `StartDate` and `EndDate` for each record

**Logic:**
- If customer data changes → Close old record, insert new record
- If customer is new → Insert with IsCurrent = 'Y'

### Step 6: Validation (06_validation)
**Quality Checks:**
- ✅ Check for NULL customer IDs
- ✅ Check for duplicate customers
- ✅ Identify products with invalid prices
- ✅ Identify stores with missing regions
- ✅ Identify sales with invalid quantities

### Step 7: Logging (07_logging)
- Process execution tracking
- Success/failure logging by layer

## 📊 Data Sources

**S3 Bucket:** `s3://capgemini-retail-etl-shiva/`

**Folders:**
- `raw/` - Latest source files
- `sftp/` - SFTP landing zone
- `archive/customers/` - Archived customer files

**File Naming Convention:**
- `customers_src_DDMMYYYYHHMMSS.csv`
- `products_src_DDMMYYYYHHMMSS.csv`
- `stores_src_DDMMYYYYHHMMSS.csv`
- `sales_transactions_src_DDMMYYYYHHMMSS.csv`

## 🚀 How to Run

### Prerequisites
- Databricks workspace with Unity Catalog enabled
- S3 access configured
- Catalog: `capgeminipro`

### Execution Order

1. **Archive old files**
   ```
   Run: 01_archival
   ```

2. **Load Bronze Layer**
   ```
   Run: 02_bronze_load
   ```

3. **Transform to Silver Layer**
   ```
   Run: 03_silver_transform
   ```

4. **Create Gold Layer**
   ```
   Run: 04_gold_load
   ```

5. **Apply SCD Type 2**
   ```
   Run: 05_scd2_merge
   ```

6. **Validate Data Quality**
   ```
   Run: 06_validation
   ```

7. **Log Process**
   ```
   Run: 07_logging
   ```

## 🔍 Key Features

✅ **Medallion Architecture** - Bronze → Silver → Gold layers  
✅ **Delta Lake Format** - ACID transactions, time travel  
✅ **Data Quality Checks** - Validation and rejection of bad data  
✅ **SCD Type 2** - Historical change tracking for customers  
✅ **File Archival** - Automated cleanup of processed files  
✅ **Unity Catalog** - Centralized data governance  

## 🛠️ Technologies Used

- **Databricks** - Unified analytics platform
- **Delta Lake** - Storage layer with ACID properties
- **Unity Catalog** - Data governance
- **Apache Spark SQL** - Data processing
- **Python** - Scripting and automation
- **AWS S3** - Cloud storage

## 📝 Data Schemas

### Bronze Schema
- All columns loaded as strings from CSV
- No data type enforcement

### Silver Schema
**Customers:** CustomerID (INT), CustomerName (STRING), Email (STRING), City (STRING), Address (STRING), LastUpdated (DATE)

**Products:** ProductID (INT), ProductName (STRING), Category (STRING), UnitPrice (DECIMAL)

**Stores:** StoreID (INT), StoreName (STRING), Region (STRING)

**Sales:** TransactionID (INT), StoreID (INT), ProductID (INT), CustomerID (INT), Quantity (INT), TransactionDate (DATE)

### Gold Schema
**dim_customers:** CustomerSK (BIGINT), CustomerID (INT), CustomerName, Email, City, Address, StartDate, EndDate, IsCurrent

**dim_products:** ProductSK (BIGINT), ProductID (INT), ProductName, Category, UnitPrice

**dim_stores:** StoreSK (BIGINT), StoreID (INT), StoreName, Region

**fact_sales:** SalesSK (BIGINT), TransactionID, StoreID, ProductID, CustomerID, Quantity, TransactionDate
