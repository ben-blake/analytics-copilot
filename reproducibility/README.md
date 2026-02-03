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
SNOWFLAKE_ROLE=SYSADMIN
SNOWFLAKE_WAREHOUSE=COMPUTE_WH
SNOWFLAKE_DATABASE=ANALYTICS_COPILOT
SNOWFLAKE_SCHEMA=RAW
```

### 2.2 Database Initialization
Run the setup script to create the necessary Database, Schemas (`RAW`, `SILVER`, `METADATA`), and Stages.

```bash
python reproducibility/setup_snowflake.py
```
*This script executes SQL commands to set up the environment structure.*

## 3. Data Ingestion

### 3.1 Download Datasets
1.  **Olist E-Commerce**: Download from [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and unzip into `data/olist/`.
2.  **Superstore Sales**: Download from [Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) and save to `data/superstore/`.

### 3.2 Load Data
Run the ingestion script to upload CSVs to Snowflake Internal Stages and copy them into tables.

```bash
python reproducibility/ingest_data.py
```
*   **Olist**: Loads 9 CSVs into `RAW.OLIST_*` tables.
*   **Superstore**: Loads 1 CSV into `RAW.SUPERSTORE_SALES`.
*   **Primary Keys**: The script applies constraints (primary/foreign keys) locally via DDL execution.

## 4. Metadata & Vector Indexing

### 4.1 Generate Semantic Metadata
We use an LLM to generate business descriptions for each table and column.

```bash
python reproducibility/build_metadata.py
```
*Input:* Raw Schema Information Schema.
*Output:* Populates `METADATA.TABLE_DESCRIPTIONS` with JSON descriptions.

### 4.2 Build Vector Index
Index the metadata using Snowflake Cortex Search (or vector tables).

```bash
python reproducibility/index_vectors.py
```
*   Embeds table descriptions.
*   Embeds "Golden Query" examples from `reproducibility/golden_queries.json`.

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
python reproducibility/generate_golden_set.py
```
*Uses Cortex LLM to halluncinate questions based on schema, then human verifies them.*

### 6.2 Run Benchmarks
Execute the evaluation suite against the Golden Dataset.

```bash
python reproducibility/evaluate.py
```

**Metrics Reported:**
*   **Execution Accuracy (EX):** % of queries that execute successfully.
*   **Result Match:** % of queries returning results matching the ground truth.
*   **Latency:** Average time per query.
