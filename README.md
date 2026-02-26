# Analytics Copilot: Ask Questions About Your Data

## Team Members
*   **Ben Blake** (GenAI & Backend Lead) - [@ben-blake](https://github.com/ben-blake)
*   **Tina Nguyen** (Data & Frontend Lead) - [@tinana2k](https://github.com/tinana2k)

## Problem Statement & Objectives
**Problem:** Non-technical users often depend on data analysts to write SQL queries or build dashboards, which creates bottlenecks and slows down decision-making processes.

**Objective:** To build an AI-Powered Data Analytics Copilot that enables self-service analytics. The system will allow business users to ask questions about their data in plain English and receive accurate answers, SQL queries, result tables, and visualizations, all while maintaining guardrails and providing clear explanations.

## System Architecture
*   **Natural Language Processing:** LLM translates natural-language questions into SQL (or dataframe operations).
*   **Context Awareness:** RAG (Retrieval-Augmented Generation) over metadata (schema, column definitions, business glossary, example questions) to reduce hallucinations and errors.
*   **Output:** The system generates the query, a summary of results, and an explanation. It can also generate optional charts from the results.
*   **Interface:** Interactive web application for uploading datasets and querying.

![Architecture Diagram](./docs/architecture.png)

## Dataset Links
1.  **Brazilian E-Commerce Public Dataset by Olist (Kaggle)**
    *   **Link:** [https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
    *   **Description:** Real commercial data containing 100k orders across 9 relational tables (customers, orders, order_items, payments, products, etc.).
    *   **Usage:** This will serve as our primary "Enterprise" demo database. We will ingest the CSVs into Snowflake, defining proper Primary/Foreign keys to test the system's ability to handle joins and complex schema relationships.

2.  **Superstore Sales Dataset (Kaggle)**
    *   **Link:** [https://www.kaggle.com/datasets/vivek468/superstore-dataset-final](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)
    *   **Description:** A classic retail dataset covering 4 years of sales data. Can be treated as a flat table or normalized into Orders/Products/Customers.
    *   **Usage:** A simpler baseline dataset for initial system testing and latency benchmarking before moving to the complex Olist schema.

3.  **Spider: Yale Semantic Parsing and Text-to-SQL Challenge (Hugging Face)**
    *   **Link:** [https://huggingface.co/datasets/xlangai/spider](https://huggingface.co/datasets/xlangai/spider)
    *   **Description:** A large-scale, complex, cross-domain text-to-SQL dataset annotated by 11 Yale students.
    *   **Usage:** We will not use this for "training" in the traditional sense, but we will index high-quality SQL patterns from Spider into our Vector Database (RAG). This allows our Copilot to retrieve "few-shot" examples of complex SQL syntax (like nested subqueries or HAVING clauses) to guide the LLM.

## Related Work (NeurIPS 2025 Paper References)
1.  **Chatting With Your Data: LLM-Enabled Data Transformation for Enterprise Text to SQL** (NeurIPS 2025)
    *   **Link:** [https://neurips.cc/virtual/2025/132553](https://neurips.cc/virtual/2025/132553)
    *   **Summary:** This paper introduces MAIA, a system that addresses the "schema gap" in enterprise databases. It transforms fragmented, technical database schemas into semantically rich, logic-aligned representations that LLMs can easier understand.
    *   **Project Integration:** We will adapt their "Schema Abstraction" technique. Instead of feeding raw DDL to our LLM, we will build a "Semantic Metadata Layer" in Snowflake that maps cryptic column names to business definitions, significantly improving Text-to-SQL accuracy.

2.  **OmniSQL: Synthesizing High-quality Text-to-SQL Data at Scale** (NeurIPS 2025)
    *   **Link:** [https://arxiv.org/abs/2503.02240](https://arxiv.org/abs/2503.02240)
    *   **Summary:** To solve the data scarcity problem, OmniSQL proposes a pipeline to generate over 2.5 million high-quality synthetic Text-to-SQL pairs. It demonstrates that models trained on this synthetic data can outperform those trained on human-annotated data.
    *   **Project Integration:** We will use a simplified version of their synthesis pipeline to "stress test" our system. We will generate synthetic user questions based on our specific schema (Olist) to create a robust evaluation set (Golden Q&A pairs) for measuring our Copilot's accuracy.

3.  **X-SQL: Expert Schema Linking and Understanding of Text-to-SQL with Multi-LLMs** (NeurIPS 2025)
    *   **Link:** [https://arxiv.org/abs/2509.05899](https://arxiv.org/abs/2509.05899)
    *   **Summary:** This work decomposes the Text-to-SQL task, using specialized "Expert" agents for Schema Linking (finding relevant tables) and Schema Understanding. It achieves state-of-the-art results by preventing the LLM from getting overwhelmed by irrelevant table info.
    *   **Project Integration:** This validates our architectural choice to use RAG for schema retrieval. We will implement a "Schema Linker" step in our pipeline that filters the 9 Olist tables down to the relevant 2-3 before the "SQL Generation" step, reducing token usage and hallucination risks.

## Repository Structure
```
analytics-copilot/
├── src/
│   ├── agents/          # Schema Linker, SQL Generator, Validator
│   │   ├── schema_linker.py
│   │   ├── sql_generator.py
│   │   └── validator.py
│   ├── utils/           # Snowflake connection, visualization
│   │   ├── snowflake_conn.py
│   │   └── viz.py
│   └── app.py           # Streamlit chat interface
├── scripts/             # Data ingestion, metadata builder, evaluation
│   ├── ingest_data.py
│   ├── build_metadata.py
│   ├── generate_golden.py
│   └── evaluate.py
├── snowflake/           # SQL DDL scripts (setup, tables, Cortex Search)
│   ├── 01_setup.sql
│   ├── 02_olist_tables.sql
│   ├── 03_superstore.sql
│   ├── 04_metadata.sql
│   └── 05_cortex_search.sql
├── data/                # CSV data, golden queries, evaluation results
│   ├── olist/           # Brazilian E-Commerce CSVs
│   ├── superstore/      # Superstore Sales CSV (optional)
│   └── golden_queries.json
├── docs/                # Architecture diagrams and project reports
│   ├── architecture.png
│   ├── architecture.mmd # Mermaid source for architecture diagram
│   └── reports/         # Phase reports and proposal (PDF + LaTeX)
│       ├── proposal.pdf
│       ├── proposal.tex
│       ├── phase2_report.pdf
│       └── phase2_report.tex
├── reproducibility/     # Setup instructions and troubleshooting guide
│   └── README.md
├── CONTRIBUTIONS.md
├── requirements.txt
└── .env.example
```

## Setup Instructions

### 1. Clone Repository
```bash
git clone https://github.com/ben-blake/analytics-copilot.git
cd analytics-copilot
```

### 2. Set Up Python Environment
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Configure Snowflake Connection
Create a `.env` file in the project root:
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

### 4. Download Datasets
Download the following datasets from Kaggle and place them in the `data/` directory:
1. **Olist E-Commerce** *(required)*: [Download from Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and unzip into `data/olist/`
2. **Superstore Sales** *(optional)*: [Download from Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) and save to `data/superstore/` — used for baseline benchmarking only; the main system focuses on Olist

### 5. Run Setup Scripts
Execute the setup SQL scripts to create database, schemas, and tables:
```bash
# Run each SQL script in order using Snowflake CLI or SnowSQL
snowsql -f snowflake/01_setup.sql
snowsql -f snowflake/02_olist_tables.sql
snowsql -f snowflake/03_superstore.sql
snowsql -f snowflake/04_metadata.sql
snowsql -f snowflake/05_cortex_search.sql
```

Or execute all at once via the connection utility (if implemented in your environment).

## Usage Instructions

### Data Ingestion
Load CSV data into Snowflake tables:
```bash
python scripts/ingest_data.py
```
This script:
- Uploads CSVs to Snowflake internal stages
- Copies data into Olist tables (`RAW.CUSTOMERS`, `RAW.ORDERS`, `RAW.ORDER_ITEMS`, etc.) and optionally `RAW.SUPERSTORE_SALES`
- Applies primary and foreign key constraints

### Build Semantic Metadata
Generate business-friendly descriptions for tables and columns using Cortex LLM:
```bash
python scripts/build_metadata.py
```
Output: Populates `METADATA.TABLE_DESCRIPTIONS` and `METADATA.COLUMN_DESCRIPTIONS` tables.

### Launch Streamlit Application
Run the interactive chat interface:
```bash
streamlit run src/app.py
```
Open your browser to `http://localhost:8501` to start asking questions about your data.

### Run Evaluation
Test the system against golden queries to measure accuracy:
```bash
# Generate golden query dataset (optional - already provided)
python scripts/generate_golden.py

# Run evaluation benchmarks
python scripts/evaluate.py
```
Metrics reported: Execution Accuracy (EX), Result Match, and Latency.

## Features
- **Natural Language to SQL**: Ask questions in plain English, get accurate SQL queries against the Olist Brazilian E-Commerce dataset
- **Context-Aware RAG**: Retrieves relevant schema metadata via Cortex Search to reduce hallucinations
- **Multi-Agent Pipeline**: Schema Linker filters relevant tables, SQL Generator creates queries, Validator auto-corrects errors
- **Auto-Visualization**: Automatically generates bar, line, or scatter charts based on query results
- **Self-Correction**: Validator retries up to 3 times with error feedback when SQL fails
- **Snowflake Cortex Integration**: Uses Cortex LLM (`llama3.1-70b`) and Cortex Search for enterprise-grade AI capabilities
