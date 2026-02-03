# Analytics Copilot: Ask Questions About Your Data in Plain English

## Team Members
*   **Ben Blake** (GenAI & Backend Lead) - @ben-blake
*   **Tina Nguyen** (Data & Frontend Lead) - @tinana2k

## Problem Statement & Objectives
**Problem:** Non-technical users often depend on data analysts to write SQL queries or build dashboards, which creates bottlenecks and slows down decision-making processes.

**Objective:** To build an AI-Powered Data Analytics Copilot that enables self-service analytics. The system will allow business users to ask questions about their data in plain English and receive accurate answers, SQL queries, result tables, and visualizations, all while maintaining guardrails and providing clear explanations.

## System Architecture
*   **Natural Language Processing:** LLM translates natural-language questions into SQL (or dataframe operations).
*   **Context Awareness:** RAG (Retrieval-Augmented Generation) over metadata (schema, column definitions, business glossary, example questions) to reduce hallucinations and errors.
*   **Output:** The system generates the query, a summary of results, and an explanation. It can also generate optional charts from the results.
*   **Interface:** Interactive web application for uploading datasets and querying.

*(Architecture diagram to be added in `docs/` folder)*

## Dataset Links
*(To be populated with 3 datasets aligned with the project theme)*

## Related Work (Paper References)
*(To be populated with 3 relevant papers from NeurIPS 2025)*

## Repository Structure
*   `/proposal`: Contains the detailed PDF project proposal.
*   `/docs`: System diagrams, design notes, and project updates.
*   `/reproducibility`: Instructions on how to reproduce the data, code, and experiments.
