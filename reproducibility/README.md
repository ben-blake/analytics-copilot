# Reproducibility Guide

This guide details how to reproduce the **Analytics Copilot** environment, ingest data, run the pipeline, and execute evaluation benchmarks.

## 1. Prerequisites

### Software & Tools
*   **Python 3.10+**: Recommended to use a virtual environment (`venv` or `conda`).
*   **Snowflake Account**: A standard trial account is sufficient. You need `ACCOUNTADMIN` or `SYSADMIN` privileges to set up databases and stages.
*   **Git**: To clone this repository.

### Python Environment
Install dependencies using the provided requirements file:
```bash
cd analytics-copilot
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

*(Note: `requirements.txt` will be populated in the root directory with `snowflake-snowpark-python`, `streamlit`, `altair`, `pandas`, `python-dotenv`, etc.)*

## 2. Snowflake Setup

### 2.1 Configuration
Create a `.env` file in the project root with your credentials:
```ini
SNOWFLAKE_ACCOUNT=your_account_identifier
SNOWFLAKE_USER=your_username
SNOWFLAKE_PASSWORD=your_password
SNOWFLAKE_PRIVATE_KEY_PATH=./rsa_key.p8
SNOWFLAKE_ROLE=TRAINING_ROLE
SNOWFLAKE_WAREHOUSE=COPILOT_WH
SNOWFLAKE_DATABASE=ANALYTICS_COPILOT
SNOWFLAKE_SCHEMA=RAW
```

### 2.2 Database Initialization
Run the setup SQL scripts to create the necessary Database, Schemas (`RAW`, `METADATA`), and File Formats.

```bash
# Using SnowSQL (recommended)
snowsql -f snowflake/01_setup.sql
snowsql -f snowflake/02_olist_tables.sql
snowsql -f snowflake/03_superstore.sql
snowsql -f snowflake/04_metadata.sql
snowsql -f snowflake/05_cortex_search.sql
```

**Expected Output:**
```
Database ANALYTICS_COPILOT successfully created.
Schema RAW successfully created.
Schema METADATA successfully created.
File format CSV_FORMAT successfully created.
```

**What This Does:**
- Creates `ANALYTICS_COPILOT` database
- Creates `RAW` schema for ingested data
- Creates `METADATA` schema for semantic descriptions
- Defines Olist tables (9 tables with PK/FK constraints)
- Defines Superstore table
- Sets up metadata tables for semantic layer
- Creates Cortex Search service for RAG retrieval

## 3. Data Ingestion

### 3.1 Download Datasets
1.  **Olist E-Commerce** *(required)*: Download from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and unzip into `data/olist/`.
2.  **Superstore Sales** *(optional)*: Download from [Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) and save to `data/superstore/`. The main system focuses on Olist; Superstore is included for baseline comparison only.

### 3.2 Load Data
Run the ingestion script to upload CSVs to Snowflake Internal Stages and copy them into tables.

```bash
python scripts/ingest_data.py
```

**CSV to Table Mappings:**

**Olist (9 tables):**
- `olist_customers_dataset.csv` → `RAW.CUSTOMERS`
- `olist_geolocation_dataset.csv` → `RAW.GEOLOCATION`
- `olist_order_items_dataset.csv` → `RAW.ORDER_ITEMS`
- `olist_order_payments_dataset.csv` → `RAW.ORDER_PAYMENTS`
- `olist_order_reviews_dataset.csv` → `RAW.ORDER_REVIEWS`
- `olist_orders_dataset.csv` → `RAW.ORDERS`
- `olist_products_dataset.csv` → `RAW.PRODUCTS`
- `olist_sellers_dataset.csv` → `RAW.SELLERS`
- `product_category_name_translation.csv` → `RAW.PRODUCT_CATEGORY_TRANSLATION`

**Superstore (1 table, optional):**
- `Sample - Superstore.csv` → `RAW.SUPERSTORE_SALES`

**Expected Output:**
```
Uploading data/olist/olist_customers_dataset.csv to stage...
Copying into RAW.CUSTOMERS...
Successfully loaded 99441 rows into RAW.CUSTOMERS
...
Ingestion complete! All tables loaded successfully.
```

**Note:** The DDL scripts already define primary and foreign key constraints. This ingestion script focuses on data loading.

## 4. Metadata & Vector Indexing

### 4.1 Generate Semantic Metadata
We use Snowflake Cortex LLM to generate business-friendly descriptions for each table and column.

```bash
python scripts/build_metadata.py
```

**What This Does:**
- Queries `INFORMATION_SCHEMA` for all tables and columns in `RAW` schema
- Uses Cortex `COMPLETE()` function to generate semantic descriptions
- Populates `METADATA.TABLE_DESCRIPTIONS` and `METADATA.COLUMN_DESCRIPTIONS`

**Input:** Raw schema metadata from `INFORMATION_SCHEMA.TABLES` and `INFORMATION_SCHEMA.COLUMNS`

**Output Sample:**
```
Processing table: RAW.CUSTOMERS
Generated description: "Customer master table containing unique customer IDs and location information"
Inserted 9 tables and 87 columns into metadata tables.
```

### 4.2 Cortex Search Service
The Cortex Search service is automatically created by `snowflake/05_cortex_search.sql`. It indexes:
- Table and column descriptions from the metadata tables
- Golden query examples from `data/golden_queries.json`

**Verify the service is running:**
```sql
SHOW CORTEX SEARCH SERVICES IN SCHEMA METADATA;
```

You should see `SCHEMA_SEARCH_SERVICE` with status `READY`.

## 5. Running the Application

Launch the Streamlit interface locally (connected to Snowflake):

```bash
streamlit run src/app.py
```

Or deploy to **Streamlit in Snowflake (SiS)**:
1.  Copy contents of `src/app.py` into a new Streamlit App in Snowflake.
2.  Add necessary packages (`snowflake-snowpark-python`) in the package selector.

## 6. Evaluation

### 6.1 Generate Golden Dataset
(Optional) To regenerate the test set:
```bash
python scripts/generate_golden.py
```

**What This Does:**
- Uses Cortex LLM to generate realistic user questions based on schema metadata
- Creates corresponding SQL queries and expected results
- Outputs to `data/golden_queries.json`

**Note:** A pre-generated golden dataset is already provided in `data/golden_queries.json` with 50 validated question-SQL pairs across easy, medium, and hard difficulty levels — all targeting the Olist dataset.

### 6.2 Run Benchmarks
Execute the evaluation suite against the Golden Dataset.

```bash
python scripts/evaluate.py
```

**Metrics Reported:**
- **Execution Accuracy (EX):** % of queries that execute successfully without errors
- **Semantic Similarity:** Cosine similarity between generated and ground truth SQL
- **Result Match:** % of queries returning results matching the expected output
- **Latency:** Average time per query (schema linking + SQL generation + validation)

**Expected Output:**
```
Running evaluation on 50 golden queries...
===========================================
Execution Accuracy: ~75-80%
Average Latency: ~5-8s per query
===========================================
Results saved to data/evaluation_results.json
```

## 7. Troubleshooting

### Common Issues and Solutions

#### 1. Missing `.env` File
**Error:** `KeyError: 'SNOWFLAKE_ACCOUNT'` or `FileNotFoundError: .env file not found`

**Solution:**
```bash
# Create .env file in project root with your credentials
cat > .env << EOF
SNOWFLAKE_ACCOUNT=your_account_identifier
SNOWFLAKE_USER=your_username
SNOWFLAKE_PRIVATE_KEY_PATH=./rsa_key.p8
SNOWFLAKE_ROLE=TRAINING_ROLE
SNOWFLAKE_WAREHOUSE=COPILOT_WH
SNOWFLAKE_DATABASE=ANALYTICS_COPILOT
SNOWFLAKE_SCHEMA=RAW
EOF
```

#### 2. Cortex Not Enabled
**Error:** `SQL compilation error: Cortex functions are not enabled for this account`

**Solution:**
- Contact Snowflake support to enable Cortex features for your account
- Alternatively, use a Snowflake trial account which includes Cortex by default
- Verify Cortex availability: `SELECT SNOWFLAKE.CORTEX.COMPLETE('llama3.1-8b', 'test');`

#### 3. Data Files Not Found
**Error:** `FileNotFoundError: [Errno 2] No such file or directory: 'data/olist/olist_customers_dataset.csv'`

**Solution:**
```bash
# Download datasets from Kaggle
# 1. Olist: https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce
# 2. Superstore: https://www.kaggle.com/datasets/vivek468/superstore-dataset-final

# Ensure directory structure:
mkdir -p data/olist
mkdir -p data/superstore

# Unzip Olist CSVs into data/olist/
# Place Superstore CSV into data/superstore/
```

#### 4. Insufficient Privileges
**Error:** `SQL access control error: Insufficient privileges to operate on database 'ANALYTICS_COPILOT'`

**Solution:**
```sql
-- Grant necessary privileges (run as ACCOUNTADMIN)
GRANT CREATE DATABASE ON ACCOUNT TO ROLE SYSADMIN;
GRANT USAGE ON WAREHOUSE COPILOT_WH TO ROLE SYSADMIN;
USE ROLE SYSADMIN;
```

#### 5. Cortex Search Service Not Ready
**Error:** `Cortex Search service SCHEMA_SEARCH_SERVICE is not ready`

**Solution:**
```sql
-- Check service status
SHOW CORTEX SEARCH SERVICES IN SCHEMA METADATA;

-- Wait for status to change from CREATING to READY (may take 1-2 minutes)
-- If stuck, drop and recreate:
DROP CORTEX SEARCH SERVICE METADATA.SCHEMA_SEARCH_SERVICE;

-- Then re-run: snowsql -f snowflake/05_cortex_search.sql
```

#### 6. Python Dependencies Missing
**Error:** `ModuleNotFoundError: No module named 'snowflake'`

**Solution:**
```bash
# Ensure virtual environment is activated
source venv/bin/activate  # or venv\Scripts\activate on Windows

# Reinstall dependencies
pip install -r requirements.txt

# Verify installation
python -c "import snowflake.snowpark; print('Success!')"
```

#### 7. Streamlit Connection Issues
**Error:** `SnowparkSQLException: Unable to connect to Snowflake`

**Solution:**
- Verify `.env` credentials are correct
- Check network connectivity to Snowflake
- Ensure warehouse is running: `ALTER WAREHOUSE COPILOT_WH RESUME IF SUSPENDED;`
- Test connection: `python -c "from src.utils.snowflake_conn import get_session; get_session()"`

#### 8. Memory Issues During Evaluation
**Error:** `MemoryError` or OOM when running evaluation

**Solution:**
```bash
# Reduce batch size in evaluate.py or run on fewer queries
# Check Snowflake warehouse size - upgrade to MEDIUM or LARGE if needed
ALTER WAREHOUSE COPILOT_WH SET WAREHOUSE_SIZE = 'MEDIUM';
```

### Getting Help

If you encounter issues not covered here:
1. Check Snowflake documentation: https://docs.snowflake.com/
2. Review logs in Snowflake Query History
3. Open an issue on the GitHub repository
4. Contact the team members listed in the main README
