# Individual Contribution Report — Tina Nguyen

**Course:** CS 5542 Big Data Analytics & Applications
**Project:** Analytics Copilot — Text-to-SQL AI Assistant
**Role:** Data & Frontend Lead
**Contribution:** 50%

---

## Personal Contribution Summary

I was responsible for the data ingestion pipeline, the Streamlit application frontend, testing, Streamlit Cloud deployment, and project documentation. My work covers the data layer (raw ingestion through golden query curation) and the full application interface from Phase 2 through the Lab 9 monitoring dashboard.

---

## Contributions by File

### Data Ingestion and Preparation
- `scripts/ingest_data.py` *(modified)* — Snowflake data ingestion pipeline: DDL execution, CSV staging via Snowpark `session.file.put()`, COPY INTO with type coercion, row count validation ([f896669](https://github.com/ben-blake/analytics-copilot/commit/f896669))
- `scripts/generate_golden.py` *(modified)* — Golden query benchmark generation (50 queries: 20 easy, 20 medium, 10 hard) ([01afb49](https://github.com/ben-blake/analytics-copilot/commit/01afb49))
- `data/instruction_dataset.json` *(new)* — Validated and curated 82-example instruction dataset for Lab 8 fine-tuning ([1cfc990](https://github.com/ben-blake/analytics-copilot/commit/1cfc990))
- `data/instruction_train.json` *(new)* — Training split (74 examples, 90%) ([83d37ee](https://github.com/ben-blake/analytics-copilot/commit/83d37ee))
- `data/instruction_val.json` *(new)* — Validation split (8 examples, 10%) ([f8d8202](https://github.com/ben-blake/analytics-copilot/commit/f8d8202))

### Streamlit Application Frontend
- `src/app.py` *(modified)* — Full Streamlit application: tabbed layout (Chat + Monitor tabs), chat history, live progress indicators, SQL expanders, Pipeline Trace expanders, result tables, Monitor dashboard (metric cards, query history, latency chart, agent step breakdown chart), graceful demo mode fallback ([a2eef7f](https://github.com/ben-blake/analytics-copilot/commit/a2eef7f))
- `.streamlit/` *(new)* — Streamlit Cloud configuration (`config.toml`, `secrets.toml.example`) and credential management ([d283a14](https://github.com/ben-blake/analytics-copilot/commit/d283a14))

### Testing
- `tests/` *(new)* — Smoke test suite (`test_smoke.py`): 13 offline tests validating imports, configuration contracts, and agent initialization ([5e8d0be](https://github.com/ben-blake/analytics-copilot/commit/5e8d0be))

### Logging
- `src/utils/logger.py` *(new)* — Structured dual file/console logging utility with tee support ([203e56b](https://github.com/ben-blake/analytics-copilot/commit/203e56b))

---

## Percentage Contribution

**50%** — Data ingestion, Streamlit frontend, visualization, testing, deployment, and documentation.

---

## Tools Used

- **Anthropic Claude Code** (`claude-opus-4-6`) — Streamlit UI iteration, debugging deployment credential issues, documentation drafting
- **Streamlit Community Cloud** — Application hosting and secrets management
- **Kaggle** — Olist Brazilian E-Commerce dataset download and preparation

---

## Reflection

My primary technical work was the Streamlit application, which evolved significantly from a simple chat interface in Phase 2 to a production-grade tabbed application with a real-time monitoring dashboard in Lab 9. The most impactful change was the Monitor tab and Pipeline Trace system — users can now see exactly which agents ran, how long each took, and whether any retries occurred, making the system transparent and debuggable. The Lab 9 deployment work also required solving a non-obvious credential problem: passing a multi-line RSA private key through Streamlit's secrets manager required specific formatting that isn't documented clearly, which I resolved and documented in the secrets template.

---

## GitHub Contribution Evidence

See repository commit history: https://github.com/ben-blake/analytics-copilot
