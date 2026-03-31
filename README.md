# Analytics Copilot: Ask Questions About Your Data

## Team Members
*   **Ben Blake** (GenAI & Backend Lead) - [@ben-blake](https://github.com/ben-blake)
*   **Tina Nguyen** (Data & Frontend Lead) - [@tinana2k](https://github.com/tinana2k)

## Problem Statement & Objectives
**Problem:** Non-technical users often depend on data analysts to write SQL queries or build dashboards, which creates bottlenecks and slows down decision-making processes.

**Objective:** To build an AI-Powered Data Analytics Copilot that enables self-service analytics. The system allows business users to ask questions about their data in plain English and receive accurate answers, SQL queries, result tables, and visualizations — all while maintaining guardrails and providing clear explanations.

## System Architecture

![Architecture Diagram](./docs/architecture.png)

The full pipeline:

**Data Sources → Multimodal Knowledge Base → Retrieval Pipeline → Domain-Adapted Model → AI Agent Reasoning Layer → Snowflake Data Warehouse → Application Interface → Monitoring & Deployment**

1. **Data Sources** — Olist Brazilian E-Commerce CSVs (9 tables, ~100k orders)
2. **Knowledge Base** — `METADATA.TABLE_DESCRIPTIONS`: LLM-generated column descriptions, synonyms, and sample values indexed by Cortex Search
3. **Retrieval Pipeline** — Schema Linker with Cortex Search (semantic), ILIKE fallback (lexical), FK-partner supplementation
4. **Domain-Adapted Model** — CodeLlama-7B-Instruct fine-tuned via LoRA/QLoRA on 82 Olist-specific instruction examples; Cortex `llama3.1-70b` as the production path
5. **AI Agent Reasoning** — SQL Generator with structured prompts and few-shot examples; Validator with self-correction loop (up to 3 retries); pipeline trace instrumentation
6. **Snowflake Warehouse** — `ANALYTICS_COPILOT.RAW` (9 tables) + `METADATA` schema; all inference runs inside Snowflake Cortex
7. **Application Interface** — Streamlit chat UI with tabbed layout, auto-visualization, inline pipeline traces, and Monitor dashboard

## Dataset

**Brazilian E-Commerce Public Dataset by Olist**
- **Link:** [https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Scale:** ~100,000 orders across 9 relational tables, 2016–2018
- **Usage:** Primary demo database. CSVs are ingested into Snowflake `RAW` schema with PK/FK constraints.

## Repository Structure
```
analytics-copilot/
├── src/
│   ├── agents/                    # Three-agent pipeline
│   │   ├── schema_linker.py       # Cortex Search RAG + FK supplementation
│   │   ├── sql_generator.py       # LLM prompt engineering + few-shot examples
│   │   └── validator.py           # EXPLAIN-based self-correction loop
│   ├── utils/
│   │   ├── snowflake_conn.py      # Snowpark connection (RSA/password, st.secrets fallback)
│   │   ├── config.py              # Config loader backed by config.yaml
│   │   ├── trace.py               # PipelineTrace instrumentation (Lab 9)
│   │   ├── logger.py              # Dual file/console structured logging
│   │   └── viz.py                 # Auto-visualization (bar, line, scatter)
│   └── app.py                     # Streamlit app (Chat + Monitor tabs)
├── scripts/
│   ├── ingest_data.py             # Snowflake CSV ingestion via Snowpark
│   ├── build_metadata.py          # LLM-generated semantic metadata
│   ├── generate_golden.py         # Golden query benchmark generation
│   ├── evaluate.py                # 50-query evaluation harness
│   ├── evaluate_adaptation.py     # Baseline vs. LoRA comparison (Lab 8)
│   ├── create_instruction_dataset.py  # 82-example Alpaca dataset (Lab 8)
│   ├── fine_tune.py               # LoRA/QLoRA fine-tuning script (Lab 8)
│   └── api_server.py              # FastAPI model server for fine-tuned model (Lab 8)
├── snowflake/                     # SQL DDL scripts
│   ├── 01_setup.sql               # Database, schemas, warehouse, file format
│   ├── 02_olist_tables.sql        # 9 RAW tables with PK/FK constraints
│   ├── 03_superstore.sql          # Optional Superstore table
│   ├── 04_metadata.sql            # TABLE_DESCRIPTIONS and COLUMN_DESCRIPTIONS
│   └── 05_cortex_search.sql       # Cortex Search service
├── notebooks/
│   └── fine_tune_colab.ipynb      # LoRA fine-tuning notebook (Colab T4 GPU)
├── data/
│   ├── olist/                     # Brazilian E-Commerce CSVs (download from Kaggle)
│   ├── golden_queries.json        # 50 golden queries (20 easy / 20 medium / 10 hard)
│   ├── instruction_dataset.json   # 82-example instruction dataset for fine-tuning
│   ├── instruction_train.json     # Training split (74 examples)
│   └── instruction_val.json       # Validation split (8 examples)
├── tests/
│   └── test_smoke.py              # 13 offline smoke tests (no Snowflake required)
├── reproducibility/
│   ├── reproduce.sh               # Single-command full pipeline orchestration
│   └── README.md                  # Detailed setup and troubleshooting guide
├── artifacts/                     # Evaluation outputs and fine-tuned model adapter
├── logs/                          # Pipeline run logs
├── docs/
│   ├── architecture.png           # System architecture diagram
│   ├── architecture.mmd           # Mermaid source
│   └── reports/                   # Phase reports (PDF + LaTeX)
├── .streamlit/
│   ├── config.toml                # Streamlit theme and server config
│   └── secrets.toml.example       # Credential template for Streamlit Cloud
├── config.yaml                    # All runtime parameters (model, seed, limits, paths)
├── requirements.txt               # Pinned Python dependencies
└── .env.example                   # Credential template for local development
```

## Setup Instructions

### Option A — Single-Command Reproduction (Recommended)

```bash
git clone https://github.com/ben-blake/analytics-copilot.git
cd analytics-copilot
cp .env.example .env        # fill in Snowflake credentials
bash reproducibility/reproduce.sh
```

Four modes are available:

| Mode | Command | Description |
|------|---------|-------------|
| Full pipeline | `reproduce.sh` | Installs deps, ingests data, builds metadata, runs evaluation |
| Offline smoke tests | `reproduce.sh --smoke` | Validates imports and config; no Snowflake required |
| Tests only | `reproduce.sh --test` | Runs smoke test suite |
| Evaluation only | `reproduce.sh --eval` | Runs 50-query benchmark against existing Snowflake setup |

### Option B — Manual Setup

#### 1. Clone and install

```bash
git clone https://github.com/ben-blake/analytics-copilot.git
cd analytics-copilot
python -m venv venv && source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

#### 2. Configure credentials

```bash
cp .env.example .env
```

Edit `.env` with your Snowflake credentials:

```ini
SNOWFLAKE_ACCOUNT=your_account_identifier
SNOWFLAKE_USER=your_username
SNOWFLAKE_PRIVATE_KEY_PATH=./rsa_key.p8
SNOWFLAKE_ROLE=TRAINING_ROLE
SNOWFLAKE_WAREHOUSE=COPILOT_WH
SNOWFLAKE_DATABASE=ANALYTICS_COPILOT
SNOWFLAKE_SCHEMA=RAW
```

#### 3. Download dataset

Download the [Olist E-Commerce dataset from Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and unzip into `data/olist/`.

#### 4. Initialize Snowflake

```bash
snowsql -f snowflake/01_setup.sql
snowsql -f snowflake/02_olist_tables.sql
snowsql -f snowflake/04_metadata.sql
snowsql -f snowflake/05_cortex_search.sql
```

#### 5. Ingest data and build metadata

```bash
python scripts/ingest_data.py
python scripts/build_metadata.py
```

#### 6. Launch application

```bash
streamlit run src/app.py
```

Open `http://localhost:8501` to start querying.

## Usage

### Chat Interface

Type any natural language question about the Olist dataset. Examples:
- *"What is the average delivery time by customer state?"*
- *"Which product categories generate the most revenue?"*
- *"Show me the top 10 sellers by order volume this year."*

Each response includes the generated SQL (collapsible), a result table, an auto-selected visualization, and a **Pipeline Trace** expander showing per-agent timing and status.

### Monitor Tab

The Monitor tab tracks live session analytics: total queries, success rate, average latency, retry count, a per-query latency chart, and an agent step breakdown chart for identifying bottlenecks.

### Evaluation

```bash
# Run 50-query golden benchmark
python scripts/evaluate.py

# Compare baseline vs. LoRA fine-tuned model (requires running api_server.py)
python scripts/evaluate_adaptation.py
```

### Smoke Tests (Offline)

```bash
python -m pytest tests/test_smoke.py -v
```

Validates imports, configuration contracts, and agent initialization without a Snowflake connection.

## Evaluation Results

| Phase | Queries | Accuracy | Avg Latency | Notes |
|-------|---------|----------|-------------|-------|
| Project 2 baseline | 50 | ~75% | 5–8s | Prompt engineering only |
| Lab 7 (reproducibility hardening) | 50 | **100%** | 17.5s | FK supplementation + isolation filter |
| Lab 8 LoRA vs. Baseline | 15 | 100% vs. 93% | 7.4s vs. 20.2s | Domain-adapted CodeLlama-7B |

## Domain Adaptation (Lab 8)

A LoRA-adapted CodeLlama-7B-Instruct model is available as an alternative inference backend:

```bash
# Start the model server (requires GPU; tested on Colab T4)
python scripts/api_server.py

# Fine-tune from scratch (optional; pre-trained adapter in artifacts/)
# Open notebooks/fine_tune_colab.ipynb in Google Colab
```

The fine-tuned model improves qualified table name usage from 87% → 100% and runs 2.7× faster than the zero-shot baseline.

## Deployment

The application is deployed on **Streamlit Community Cloud**. To deploy your own instance:

1. Fork the repository and connect it to Streamlit Cloud.
2. Copy `.streamlit/secrets.toml.example` to `.streamlit/secrets.toml` and fill in credentials.
3. Add the secrets via the Streamlit Cloud dashboard (Settings → Secrets).

If no Snowflake credentials are configured, the application starts in **demo mode** with a "Disconnected" banner rather than crashing.

**Live app:** https://cs5542-analytics-copilot.streamlit.app

## Related Work

1. **X-SQL: Expert Schema Linking and Understanding of Text-to-SQL with Multi-LLMs** (NeurIPS 2025) — [arXiv:2509.05899](https://arxiv.org/abs/2509.05899) — validates the multi-agent decomposition architecture; FK-partner supplementation and structured error feedback are directly adopted from this work.
2. **Chatting With Your Data: LLM-Enabled Data Transformation for Enterprise Text to SQL** (NeurIPS 2025) — [link](https://neurips.cc/virtual/2025/132553) — motivates the Semantic Metadata Layer approach.
3. **OmniSQL: Synthesizing High-quality Text-to-SQL Data at Scale** (NeurIPS 2025) — [arXiv:2503.02240](https://arxiv.org/abs/2503.02240) — informs the synthetic golden query generation pipeline.

## AI Tools Used

- **Anthropic Claude Code** (`claude-opus-4-6`) — code generation, pipeline debugging, `reproduce.sh` scaffolding, instruction dataset drafting. All generated code was reviewed and validated by the team.
