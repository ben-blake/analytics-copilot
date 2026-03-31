# Individual Contribution Report — Ben Blake

**Course:** CS 5542 Big Data Analytics & Applications
**Project:** Analytics Copilot — Text-to-SQL AI Assistant
**Role:** GenAI & Backend Lead
**Contribution:** 50%

---

## Personal Contribution Summary

I was responsible for all GenAI pipeline components, Snowflake backend infrastructure, reproducibility engineering, and domain adaptation (Lab 8). My work spans the full backend stack from raw data ingestion through agent reasoning and model fine-tuning.

---

## Contributions by File

### Core Agent Pipeline
- `src/agents/schema_linker.py` *(modified)* — Schema Linker agent: Cortex Search RAG, three-level fallback (semantic → lexical → full-dump), FK-partner supplementation, dataset isolation filter ([e30d85c](https://github.com/ben-blake/analytics-copilot/commit/e30d85c))
- `src/agents/sql_generator.py` *(modified)* — SQL Generator agent: structured prompt engineering, 15-rule constraint system, few-shot examples, Cortex `llama3.1-70b` integration ([9094776](https://github.com/ben-blake/analytics-copilot/commit/9094776))

### Snowflake & Backend Infrastructure
- `src/utils/snowflake_conn.py` *(modified)* — Snowflake Snowpark connection utility with RSA/password auth and dual credential sources (`st.secrets` + `.env`) ([2976972](https://github.com/ben-blake/analytics-copilot/commit/2976972))
- `scripts/build_metadata.py` *(modified)* — LLM-driven semantic metadata generation (Cortex COMPLETE over all RAW columns) ([2608edb](https://github.com/ben-blake/analytics-copilot/commit/2608edb))
- `src/utils/config.py` *(new)* — Centralized configuration loader backed by `config.yaml` ([a55cbad](https://github.com/ben-blake/analytics-copilot/commit/a55cbad))
- `config.yaml` *(new)* — All runtime parameters: model, temperature, seed 42, top-k, retry limit, paths, logging level ([4de2add](https://github.com/ben-blake/analytics-copilot/commit/4de2add))

### Evaluation Pipeline
- `scripts/evaluate.py` *(modified)* — 50-query golden benchmark evaluation harness (execution accuracy, latency by difficulty) ([af6e851](https://github.com/ben-blake/analytics-copilot/commit/af6e851))
- `scripts/evaluate_adaptation.py` *(new)* — Side-by-side baseline vs. fine-tuned model evaluation (15 queries) ([5178307](https://github.com/ben-blake/analytics-copilot/commit/5178307))

### Reproducibility (Lab 7)
- `reproducibility/reproduce.sh` *(new)* — Single-command full pipeline orchestration (4 modes: full, --smoke, --test, --eval) ([ecbe1fb](https://github.com/ben-blake/analytics-copilot/commit/ecbe1fb))
- `requirements.txt` *(modified)* — Pinned dependency versions for reproducible environments ([89e9938](https://github.com/ben-blake/analytics-copilot/commit/89e9938))

### Domain Adaptation (Lab 8)
- `scripts/create_instruction_dataset.py` *(new)* — 82-example Alpaca-format instruction dataset (32 from golden queries + 50 augmented) ([8b1a33e](https://github.com/ben-blake/analytics-copilot/commit/8b1a33e))
- `scripts/fine_tune.py` *(new)* — LoRA/QLoRA fine-tuning script for CodeLlama-7B-Instruct ([2976c5e](https://github.com/ben-blake/analytics-copilot/commit/2976c5e))
- `notebooks/` *(new)* — Colab fine-tuning notebook (`fine_tune_colab.ipynb`) ([196759a](https://github.com/ben-blake/analytics-copilot/commit/196759a))
- `scripts/api_server.py` *(new)* — FastAPI model server exposing `/generate`, `/generate-baseline`, `/health` endpoints ([c74a3c2](https://github.com/ben-blake/analytics-copilot/commit/c74a3c2))
- `src/utils/finetuned_client.py` *(new)* — HTTP client for fine-tuned model integration into the agent pipeline ([52c448e](https://github.com/ben-blake/analytics-copilot/commit/52c448e))

### Application Enhancements (Lab 9)
- `src/utils/trace.py` *(new)* — PipelineTrace instrumentation module (per-agent timing, retry detection, session state storage) ([3b983d7](https://github.com/ben-blake/analytics-copilot/commit/3b983d7))

---

## Percentage Contribution

**50%** — Backend agents, Snowflake infrastructure, evaluation, reproducibility, and Lab 8 domain adaptation.

---

## Tools Used

- **Anthropic Claude Code** (`claude-opus-4-6`) — code generation, pipeline debugging (output buffering, SQL parser bugs, Cortex Search qualification), `reproduce.sh` scaffolding, instruction dataset drafting
- **Google Colab (T4 GPU)** — LoRA fine-tuning of CodeLlama-7B-Instruct
- **Snowflake Cortex** — LLM inference (`CORTEX.COMPLETE`), semantic search (`CORTEX.SEARCH_PREVIEW`)

---

## Reflection

My primary technical work was the three-agent pipeline that converts natural language to Snowflake SQL. The most challenging part was the accuracy improvements in Lab 7: achieving 100% on 50 queries required diagnosing why Cortex Search was dropping FK-related tables and implementing the supplementation logic. The Lab 8 fine-tuning demonstrated that even 82 training examples with LoRA produce measurable improvements — 100% qualified table names vs. 87% for the zero-shot baseline, and 2.7× faster inference — validating that domain adaptation is viable even at very small dataset scales.

---

## GitHub Contribution Evidence

See repository commit history: https://github.com/ben-blake/analytics-copilot
