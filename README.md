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
*   `/proposal`: Contains the detailed PDF project proposal.
*   `/docs`: System diagrams, design notes, and project updates.
*   `/reproducibility`: Instructions on how to reproduce the data, code, and experiments.
