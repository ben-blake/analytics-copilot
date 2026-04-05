# Analytics Copilot: Ask Questions About Your Data

**Live app:** [https://cs5542-analytics-copilot.streamlit.app](https://cs5542-analytics-copilot.streamlit.app)
**Video demo:** [https://vimeo.com/1180190697](https://vimeo.com/1180190697)

**CS 5542 Big Data Analytics & Applications · University of Missouri–Kansas City**

## Team Members
*   **Ben Blake** (GenAI & Backend Lead) - [@ben-blake](https://github.com/ben-blake)
*   **Tina Nguyen** (Data & Frontend Lead) - [@tinana2k](https://github.com/tinana2k)

---

## Project Description

Non-technical users depend on data analysts to write SQL queries, creating bottlenecks that slow decision-making. Analytics Copilot removes that barrier: type a business question in plain English, and the system translates it to Snowflake SQL, executes it, and returns a result table with an auto-selected chart — in about 7 seconds.

The core is a **three-agent pipeline** running on Snowflake Cortex:

1. **Schema Linker** — semantic RAG (Cortex Search) identifies the relevant tables from 9, with automatic FK-partner supplementation so joins are never missing
2. **SQL Generator** — sends the filtered schema to `llama3.1-70b` via a 15-rule structured prompt and few-shot example bank
3. **Validator** — runs `EXPLAIN` on the output and feeds errors back to the generator for up to 3 self-correction retries

A **LoRA-adapted CodeLlama-7B** (82 Olist-specific training examples, QLoRA 4-bit) provides an offline-deployable alternative inference path.

---

## Quick Demo

**No setup required — open the live app:**

```
https://cs5542-analytics-copilot.streamlit.app
```

The app runs in **demo mode** without Snowflake credentials — all UI components are visible. To run live queries, configure credentials (see [Setup](#setup)).

**Example questions to try:**
- *"What is the average delivery time by customer state?"*
- *"Which product categories generate the most revenue?"*
- *"Show me the top 10 sellers by total order volume."*
- *"What percentage of orders were delivered late in 2017?"*

Each response shows the generated SQL (collapsible), a result table, an auto-selected chart, and a **Pipeline Trace** panel with per-agent timing.

---

## System Architecture

![Architecture Diagram](./docs/architecture.png)

```
Kaggle CSVs (9 tables)
  → Snowflake RAW Schema (~100k rows)
  → METADATA Schema (LLM-generated column descriptions + Cortex Search index)
  → Schema Linker Agent (RAG + FK-partner supplementation)
  → SQL Generator Agent (structured prompt + few-shot examples)
  → Validator Agent (EXPLAIN + self-correction loop, up to 3 retries)
  → Streamlit Application (Chat tab + Monitor tab)
```

| Component | Technology |
|-----------|------------|
| Data warehouse | Snowflake (`ANALYTICS_COPILOT.RAW`) |
| LLM inference | Snowflake Cortex (`llama3.1-70b`) |
| Semantic search | Snowflake Cortex Search |
| Domain-adapted model | CodeLlama-7B-Instruct + LoRA/QLoRA |
| Application | Streamlit (Streamlit Community Cloud) |
| Orchestration | Python + Snowflake Snowpark |

---

## Dataset

**Brazilian E-Commerce Public Dataset by Olist** — [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (CC BY-NC-SA 4.0)

~100,000 orders across 9 relational tables, 2016–2018, ingested into `ANALYTICS_COPILOT.RAW`:

| Table | Rows | Description |
|-------|------|-------------|
| `ORDERS` | 99,441 | Order lifecycle — anchor table |
| `CUSTOMERS` | 99,441 | Demographics and location |
| `ORDER_ITEMS` | 112,650 | Line items and pricing |
| `ORDER_PAYMENTS` | 103,886 | Payment method and value |
| `ORDER_REVIEWS` | 99,224 | Star ratings and comments |
| `PRODUCTS` | 32,951 | Category and dimensions |
| `SELLERS` | 3,095 | Seller location |
| `GEOLOCATION` | 1,000,163 | Lat/lon by zip code |
| `CATEGORY_TRANSLATION` | 71 | English category names |

A semantic metadata layer (`METADATA.TABLE_DESCRIPTIONS`) stores LLM-generated column descriptions, synonyms, and sample values for every column, indexed by Cortex Search for retrieval.

---

## Repository Structure

```
analytics-copilot/
├── src/
│   ├── agents/
│   │   ├── schema_linker.py           # Cortex Search RAG + FK supplementation
│   │   ├── sql_generator.py           # Structured prompt + few-shot examples
│   │   └── validator.py               # EXPLAIN-based self-correction loop
│   ├── utils/
│   │   ├── snowflake_conn.py          # Snowpark connection (RSA / st.secrets)
│   │   ├── config.py                  # Config loader (config.yaml)
│   │   ├── trace.py                   # PipelineTrace: per-agent timing + status
│   │   ├── logger.py                  # Dual file/console structured logging
│   │   └── viz.py                     # Auto-visualization (bar, line, scatter)
│   └── app.py                         # Streamlit app — Chat + Monitor tabs
├── scripts/
│   ├── ingest_data.py                 # CSV → Snowflake RAW via Snowpark
│   ├── build_metadata.py              # LLM-generated semantic metadata
│   ├── generate_golden.py             # Golden query benchmark generation
│   ├── evaluate.py                    # 50-query evaluation harness
│   ├── evaluate_adaptation.py         # Baseline vs. LoRA comparison
│   ├── create_instruction_dataset.py  # 82-example Alpaca instruction dataset
│   ├── fine_tune.py                   # LoRA/QLoRA fine-tuning script
│   └── api_server.py                  # FastAPI server for fine-tuned model
├── snowflake/
│   ├── 01_setup.sql                   # Database, schemas, warehouse, file format
│   ├── 02_olist_tables.sql            # 9 RAW tables with PK/FK constraints
│   ├── 04_metadata.sql                # TABLE_DESCRIPTIONS table
│   └── 05_cortex_search.sql           # Cortex Search service
├── notebooks/
│   └── fine_tune_colab.ipynb          # LoRA fine-tuning notebook (Colab T4 GPU)
├── data/
│   ├── olist/                         # Olist CSVs (download from Kaggle)
│   ├── golden_queries.json            # 50 golden queries (20 / 20 / 10 easy/med/hard)
│   ├── instruction_dataset.json       # 82-example fine-tuning dataset
│   ├── instruction_train.json         # Training split (74 examples)
│   └── instruction_val.json           # Validation split (8 examples)
├── tests/
│   └── test_smoke.py                  # 13 offline smoke tests (no Snowflake required)
├── reproducibility/
│   ├── reproduce.sh                   # Single-command full pipeline script
│   └── README.md                      # Setup guide and troubleshooting
├── docs/
│   ├── architecture.png               # System architecture diagram
│   ├── architecture.mmd               # Mermaid source
│   └── reports/                       # Phase reports, poster, presentation
├── .streamlit/
│   ├── config.toml                    # Streamlit theme and server config
│   └── secrets.toml.example           # Credential template for Streamlit Cloud
├── config.yaml                        # All runtime parameters (centralized)
├── requirements.txt                   # Pinned Python dependencies
└── .env.example                       # Credential template for local development
```

---

## Setup

### Option A — Single Command (Recommended)

```bash
git clone https://github.com/ben-blake/analytics-copilot.git
cd analytics-copilot
cp .env.example .env        # fill in Snowflake credentials
bash reproducibility/reproduce.sh
```

| Mode | Command | What it does |
|------|---------|--------------|
| Full pipeline | `reproduce.sh` | Installs deps, ingests data, builds metadata, runs evaluation |
| Offline only | `reproduce.sh --smoke` | Validates imports and config — no Snowflake required |
| Tests only | `reproduce.sh --test` | Runs the 13-test smoke suite |
| Eval only | `reproduce.sh --eval` | Runs 50-query benchmark against an existing Snowflake setup |

### Option B — Manual Setup

**1. Clone and install**

```bash
git clone https://github.com/ben-blake/analytics-copilot.git
cd analytics-copilot
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

**2. Configure credentials**

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

**3. Download the dataset**

Download the [Olist dataset from Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) and unzip into `data/olist/`.

**4. Initialize Snowflake**

```bash
snowsql -f snowflake/01_setup.sql
snowsql -f snowflake/02_olist_tables.sql
snowsql -f snowflake/04_metadata.sql
snowsql -f snowflake/05_cortex_search.sql
```

**5. Ingest data and build metadata**

```bash
python scripts/ingest_data.py
python scripts/build_metadata.py
```

**6. Launch the application**

```bash
streamlit run src/app.py
```

Open [http://localhost:8501](http://localhost:8501).

---

## How to Run

### Application

```bash
streamlit run src/app.py
```

- **Chat tab** — type a natural language question, receive SQL + result table + auto-chart + Pipeline Trace
- **Monitor tab** — session-level success rate, average latency, per-query history, agent step breakdown chart

### Evaluation

```bash
# 50-query golden benchmark (easy / medium / hard)
python scripts/evaluate.py

# Baseline vs. LoRA fine-tuned model side-by-side
# (requires api_server.py running — see Domain Adaptation)
python scripts/evaluate_adaptation.py
```

### Smoke Tests (no Snowflake required)

```bash
python -m pytest tests/test_smoke.py -v
```

Validates imports, configuration contracts, and agent initialization entirely offline.

---

## Reproducibility

All runtime parameters are centralized in `config.yaml` — no magic numbers in code:

```yaml
seed: 42
llm:
  model: llama3.1-70b
  temperature: 0.0
schema_linker:
  limit: 5
sql_generator:
  max_retries: 3
```

| Control | Mechanism |
|---------|-----------|
| Pinned dependencies | `requirements.txt` with `==` versions |
| Deterministic LLM | `temperature: 0.0`, `seed: 42` |
| Config-driven | All parameters in `config.yaml` — no magic numbers in code |
| Structured logging | `logs/pipeline.log` + console output via `tee` |
| Offline validation | 13 smoke tests run with no Snowflake connection |
| Single-command | `reproducibility/reproduce.sh` orchestrates the full pipeline end-to-end |

See [`reproducibility/README.md`](reproducibility/README.md) for the full audit checklist and troubleshooting guide.

---

## Demo Instructions

### Live App (No Setup)

1. Open [https://cs5542-analytics-copilot.streamlit.app](https://cs5542-analytics-copilot.streamlit.app)
2. The app loads in **demo mode** without credentials — UI is fully functional, queries require a Snowflake connection
3. To enable live queries: add Snowflake credentials via Streamlit Cloud → Settings → Secrets (use `.streamlit/secrets.toml.example` as a template)

### Local Demo Mode

```bash
streamlit run src/app.py
```

Without a `.env` or `st.secrets`, the app starts with a "Disconnected" banner. The Monitor tab and all UI components still render.

### Example Walkthrough

1. Type: *"Which product categories generate the most revenue?"*
2. **Schema Linker** retrieves `ORDER_ITEMS`, `PRODUCTS`, `CATEGORY_TRANSLATION`
3. **SQL Generator** writes a `GROUP BY` + `SUM(PRICE)` query with a 3-table `JOIN`
4. **Validator** runs `EXPLAIN`, confirms valid, executes
5. Results appear as a table + bar chart; expand **Pipeline Trace** to see per-agent timing

---

## Evaluation Results

| Phase | Queries | Accuracy | Avg Latency | Notes |
|-------|---------|----------|-------------|-------|
| Project 2 baseline | 50 | ~75% | 5–8 s | Prompt engineering only |
| Lab 7 hardened | 50 | **100%** | 17.5 s | FK supplementation + isolation filter |
| Lab 8 LoRA | 15 | **100%** | 7.4 s | Domain-adapted CodeLlama-7B |
| Lab 8 Baseline | 15 | 93% | 20.2 s | Zero-shot CodeLlama-7B |

---

## Domain Adaptation (Lab 8)

A LoRA-adapted CodeLlama-7B-Instruct model is available as an alternative inference backend.

**Start the model server** (GPU required; tested on Colab T4):

```bash
python scripts/api_server.py
```

**Fine-tune from scratch** (optional — pre-trained adapter in `artifacts/`):

Open `notebooks/fine_tune_colab.ipynb` in Google Colab and run all cells.

The fine-tuned model vs. zero-shot baseline:

| Metric | Baseline | LoRA Fine-tuned |
|--------|----------|-----------------|
| Valid SQL generated | 93% | **100%** |
| Qualified table names | 87% | **100%** |
| Avg inference latency | 20.2 s | **7.4 s** |

---

## Deployment

The application is deployed on **Streamlit Community Cloud**.

To deploy your own instance:

1. Fork the repository and connect it to [Streamlit Cloud](https://streamlit.io/cloud)
2. Copy `.streamlit/secrets.toml.example` → `.streamlit/secrets.toml` and fill in credentials
3. Add secrets in the Streamlit Cloud dashboard: **Settings → Secrets**

The app starts in demo mode if no credentials are configured, rather than crashing.

---

## Related Work

1. **X-SQL: Expert Schema Linking and Understanding of Text-to-SQL with Multi-LLMs** (NeurIPS 2025) — [arXiv:2509.05899](https://arxiv.org/abs/2509.05899) — validates multi-agent decomposition; FK-partner supplementation and structured error feedback adopted directly
2. **Chatting With Your Data: LLM-Enabled Data Transformation for Enterprise Text to SQL** (NeurIPS 2025) — motivates the Semantic Metadata Layer approach
3. **OmniSQL: Synthesizing High-quality Text-to-SQL Data at Scale** (NeurIPS 2025) — [arXiv:2503.02240](https://arxiv.org/abs/2503.02240) — informs synthetic golden query generation

---

## AI Tools Used

- **Anthropic Claude Code** (`claude-opus-4-6`) — code generation, pipeline debugging, `reproduce.sh` scaffolding, instruction dataset drafting. All generated code was reviewed and validated by the team.
