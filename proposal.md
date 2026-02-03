# CS 5542 Big Data Analytics and Applications

## Project Proposal (4 %)

**Deadline:** Friday, Feb. 6 (Midnight)
**Submission:** Team Submission
**Format:** Proposal Document (about 10 pages expected, PDF)

You are expected to build on your pre-proposal and instructor feedback to elevate your work into a true “aha!” project — a system that feels like a research prototype or startup-grade product, not a standard class assignment. You may refine, merge, or change your original project ideas if your literature review, dataset exploration, or system design motivates a stronger direction.

---

## Required Components

### Project Selection

Choose one project direction to pursue. This may be a refined or revised version of one of your original ideas.

---

### Project GitHub Repository (Required)

Create a team GitHub repository and connect it with the GitHub accounts of all team members.

Your repository must include:

* A clear README.md with:

  * Project title and team members (with GitHub usernames)
  * Problem statement and objectives
  * System architecture diagram
  * Dataset links and paper references
* A /proposal folder containing your PDF proposal
* A /docs or /design folder for system diagrams, notes, and updates
* A /reproducibility section describing how data, code, and experiments will be run

All future project updates, documentation, and artifacts should be posted and maintained in this GitHub repository.

---

### Related Work — Research Papers (with Code)

Find and cite 3 relevant papers from the NeurIPS 2025 proceedings that directly align with your chosen project.

#### How to Browse (Required)

Search using keywords derived from your project idea.

**Example keywords:**

* RAG, enterprise analytics, knowledge graphs, governance AI
* time-series forecasting, spatio-temporal modeling, multimodal learning
* fraud detection, financial risk, compliance systems
* healthcare AI, clinical prediction, explainability

#### Where to Browse

* NeurIPS 2025 Proceedings [https://neurips.cc/virtual/2025/papers.html](https://neurips.cc/virtual/2025/papers.html) Links to an external site.
* Indexed list with code and data: [https://www.paperdigest.org/2025/11/neurips-2025-papers-with-code-data/](https://www.paperdigest.org/2025/11/neurips-2025-papers-with-code-data/) Links to an external site.
* KDD 2025 Proceedings [https://dl.acm.org/doi/proceedings/10.1145/3711896](https://dl.acm.org/doi/proceedings/10.1145/3711896) Links to an external site.

#### Requirements for Each Paper

For each paper, include:

* Paper link
* Code/GitHub link
* 2–3 sentence summary
* A short explanation of how your project extends, adapts, or integrates this work into a Snowflake-centered system pipeline

---

### Data Sources (Keyword-Driven Search)

Find 3 datasets aligned with your project theme from:

* Kaggle: [https://www.kaggle.com/datasets](https://www.kaggle.com/datasets) Links to an external site.
* Hugging Face Datasets: [https://huggingface.co/datasets](https://huggingface.co/datasets) Links to an external site.
* Other open repositories (UCI ML Repository, AWS Open Data, government APIs)

#### Requirements for Each Dataset

For each dataset, include:

* Direct dataset link
* Brief description (size, modality, update frequency)
* How it will be ingested, stored, and processed in Snowflake (e.g., stages, external tables, Snowpipe, Snowpark)

---

## Proposal Structure (Be Specific)

### Objectives

* Clearly define the real-world problem
* Identify target users and decisions your system improves
* Describe the innovation layer that makes your system novel

---

### Methods, Technologies & Tools

Your proposal must describe a clear and coherent system architecture. You may use Snowflake and its ecosystem as a recommended platform, or you may design and implement an equivalent custom or alternative stack (e.g., self-hosted databases, cloud-native services, open-source pipelines).

#### Data Ingestion & Storage (Recommended: Snowflake or Equivalent)

* Snowflake stages, Snowpipe, external tables, streams/tasks or equivalent ingestion tools
* Batch vs. streaming ingestion design
* Data modeling (schemas, views, governance, access control)

#### Analytics, ML & GenAI

* Snowpark (Python/SQL/Scala) or equivalent processing frameworks (e.g., Spark, Pandas, DuckDB)
* LLMs / RAG pipelines (Snowflake Cortex, external APIs, or Hugging Face models)
* Vector embeddings and vector search (inside Snowflake or via external vector databases/services)
* Optional neuro-symbolic or rule-based validation layers

#### Infrastructure & Deployment

* Dashboards (Streamlit, Snowflake Native Apps, or web frontends)
* Cloud services (AWS / GCP / Azure) or local/on-prem deployment
* Monitoring, logging, and evaluation pipelines

---

### Expected Outcomes

* What your live demo will show (e.g., analytics dashboard, AI assistant, simulation system)
* What metrics you will evaluate (accuracy, latency, scalability, explainability, trust, governance)

---

## Final Deliverables:

* Working system prototype
* Snowflake SQL / Snowpark notebooks or equivalent scripts/notebooks for your chosen stack
* Public or private GitHub repository with full documentation and updates
* Evaluation & design report

---

## Evaluation Criteria

Strong proposals will demonstrate:

* Intentional paper and dataset selection based on project keywords
* A clear Snowflake-centered system architecture
* A reproducibility plan (code, data, SQL/Snowpark pipelines, evaluation) documented in GitHub
* A visible innovation layer (simulation, prediction, governance, causal reasoning, or structured intelligence)
* Professional structure, clarity, and technical depth appropriate for a ~10-page research-style proposal

---
