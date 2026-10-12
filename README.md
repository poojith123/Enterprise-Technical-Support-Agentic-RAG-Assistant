# Enterprise Technical Support Agentic RAG Assistant
This RAG chatbot helps to answer typical technical questions from complex large pool of technical documents with noisy data. It dynamically decides whether to retrieve documents or answer from conversation history. Answers are grounded with citations. 

The chatbot also incorporates input guardrails, LangGraph-based orchestration, LLM Gateway integration, conversational memory, document citations, and retrieval reranking to provide reliable and traceable technical answers.

# Intelligent flow diagram
```mermaid
graph TD
    %% 3-Color Dark Palette
    classDef default fill:#1E293B,stroke:#64748B,stroke-width:1.5px,color:#FFFFFF;
    classDef core fill:#0F172A,stroke:#3B82F6,stroke-width:1.5px,color:#93C5FD;
    classDef alert fill:#450A0A,stroke:#EF4444,stroke-width:1.5px,color:#FCA5A5;

    linkStyle default stroke:#94A3B8,stroke-width:1.5px;

    %% 1. USER AT THE VERY TOP
    User((User)):::default --> UI[Streamlit UI]:::default
    UI --> API[FastAPI /query]:::default
    API --> Guard{NeMo Guardrails}:::alert

    %% 2. GUARDRAILS DECISION
    Guard -->|Blocked| BlockedMsg[Blocked Screen]:::alert
    BlockedMsg -.-> UI
    Guard -->|Pass| Planner{Planner Node}:::core

    %% 3. PLANNER ROUTING
    Planner -->|Conversational| Responder[Responder Node]:::core
    Planner -->|Technical| Retriever[Retriever Node]:::default

    %% 4. INGESTION PIPELINE (Aligned horizontally to the side)
    subgraph DataPipeline ["Data Store"]
        Docs[Documents]:::default --> Embed[Gemini Embeddings]:::default --> DB[(Qdrant VectorDB)]:::default
    end

    %% Lateral retrieval link (does not pull hierarchy to top)
    Retriever <-->|Search & Fetch| DB
    Retriever --> Reranker[FlashRank Reranker]:::default
    Reranker --> Responder

    %% 5. MEMORY & RETURN
    Responder -.-> Memory[(LangGraph Memory)]:::default
    Responder -->|Return Response| UI
```
