# System Architecture Design

## Overview
This architecture leverages **Snowflake** as the central data and compute engine. The system follows a standard **Data-to-App** pipeline where unstructured and structured data is ingested into Snowflake, indexed for AI retrieval, and served to business users via a conversational interface.

## High-Level Architecture

![Architecture Diagram](architecture.png)

## Component Breakdown

### 1. Data Sources & Ingestion
*   **Sources:** External datasets (Kaggle) and user-uploaded CSVs.
*   **Ingestion:** Data is loaded into Snowflake using **Internal Stages** and **COPY INTO** commands.
*   **Storage:** Data resides in **Standard Tables** (Silver/Gold layer) optimized for analytical queries.

### 2. Snowflake Data Cloud (The Engine)
*   **Storage:** Holds the relational data and the **Vector Store** (containing embeddings of schema definitions and example queries).
*   **Cortex AI:** Provides the serverless **LLM** (e.g., Llama 3 via Snowflake Cortex) for Text-to-SQL generation and **Vector Search** capabilities.
*   **SQL Engine:** Executes the generated queries against the data warehouse.

### 3. Application Layer (The Logic)
*   **Copilot Orchestrator:** A Python-based backend (likely running in **Snowpark Container Services** or externally) that manages the RAG pipeline. It handles the "thought process": retrieving context, prompting the LLM, and handling errors.
*   **Streamlit Interface:** A web-based chat UI where users input questions and view results (tables/charts).

### 4. User Workflow
1.  **Ask:** User asks "What were total sales in 2024?"
2.  **Retrieve:** Orchestrator searches the **Vector Store** for relevant table schemas.
3.  **Generate:** Orchestrator sends the question + schema to **Cortex AI** to generate SQL.
4.  **Execute:** The generated SQL runs on the **Compute** engine.
5.  **Answer:** Results are returned to the UI for visualization.
