# AI-Powered Engineering Knowledge

## 1. Project overview

Build an AI assistant that understands an organisation's engineering
knowledge and helps software teams find information, analyse
requirements, and create technical deliverables. This project demonstrates practical enterprise AI engineering rather than just building a basic chatbot. It also gives you experience with solution architecture, agent orchestration, RAG pipelines, evaluation and observability.

### Problem statement

Software engineering teams work with large amounts of technical
information:

-   Software requirement documents (PRDs)
-   Solution architecture documents
-   API documentation
-   Technical specifications
-   Source code and GitHub repositories
-   Database schemas
-   Troubleshooting guides
-   Previous project documentation

Finding relevant information across these sources takes time.

### Proposed solution

Build an AI assistant that can:

1.  Answer technical questions using internal documents.
2.  Retrieve relevant documents using semantic search.
3.  Generate solution architecture proposals.
4.  Analyse requirements and identify missing information.
5.  Generate implementation plans and user stories.
6.  Generate SQL queries based on database documentation.
7.  Provide references and citations for generated answers.
8.  Evaluate answer quality and monitor AI performance.

### Example questions

-   What is the architecture of our customer management system?
-   Generate an architecture proposal for a new payment integration.
-   What APIs are available for customer authentication?
-   Identify technical gaps in this PRD.
-   Generate user stories from this requirement document.
-   Which documents describe our authentication strategy?

------------------------------------------------------------------------

## 2. Technology stack

|Component             | Technology           | Purpose              |
|----------------------|----------------------|----------------------|
|LLM                   |OpenAI / Azure OpenAI |Generate answers |
|RAG framework         |LangChain             |Document loading, chunking, retrieval | 
|Agent orchestration   |LangGraph             |Multi-step agent workflows | 
|Vector database       |Qdrant / Postgres + pgvector / pinecone / Chroma                |Store and search embeddings |
|Embeddings            |OpenAI Embeddings     |Convert documents into vectors |
|Observability         |LangFuse / Comet Opik              |Traces, token usage, latency, evaluations |
|Backend               |Python + FastAPI      |REST APIs |
|Frontend              |React                 |Chat interface | 
|Database              |PostgreSQL            |Users, documents, metadata |
|Containerisation      |Docker                |Local deployment |

### Recommended starting choices

-   Vector database: Qdrant / Postgres + pgvector / pinecone / Chroma.
-   RAG framework: LangChain.
-   Orchestration: LangGraph.
-   Observability: LangFuse / Comet Opik.
-   LLM: Azure OpenAI or OpenAI.

Start with Qdrant locally using Docker, then consider implementing Azure
AI Search as an alternative backend.

------------------------------------------------------------------------

## 3. High-level architecture

<img width="1200" height="657" alt="image" src="https://github.com/user-attachments/assets/8c2ecb51-5cbb-46cc-a6ee-41e07f26cecf" />

The application has two major pipelines.

### A. Document ingestion pipeline

1.  Upload documents (PDF, DOCX, Markdown, TXT).
2.  Extract document content.
3.  Clean and normalise text.
4.  Split documents into chunks.
5.  Generate embeddings.
6.  Store embeddings and metadata in Qdrant or Postgres + pgvector.
7.  Maintain document references and access permissions.

### B. Question-answering pipeline

1.  User submits a question.
2.  LangChain converts the question into an embedding.
3.  Qdrant / Postgres + pgvector retrieves relevant chunks.
4.  Apply metadata filtering and reranking.
5.  LangGraph determines the next action.
6.  The LLM generates an answer using retrieved context.
7.  Return citations and source documents.
8.  LangFuse records execution traces and performance.

------------------------------------------------------------------------

## 4. LangGraph -- Multi-agent workflow

Instead of implementing a simple RAG chain, use LangGraph to create a
workflow with specialised nodes.

### Proposed workflow

``` mermaid
flowchart TD
    A[User Question] --> B[Question Classifier]
    B --> C{Question Type}
    C -->|Technical Search| D[Document Retriever]
    C -->|Architecture| E[Architecture Agent]
    C -->|Requirements| F[Business Analyst Agent]
    C -->|SQL| G[SQL Agent]
    D --> H[Retrieval Evaluator]
    H --> I{Relevant Context?}
    I -->|No| J[Query Rewriter]
    J --> D
    I -->|Yes| K[Answer Generator]
    E --> K
    F --> K
    G --> K
    K --> L[Response Evaluator]
    L --> M[Final Answer + Citations]
```

### Suggested agents and nodes

  |Agent                    |Responsibility        |
  |-------------------------|----------------------|
  |Question Classifier      |Understand user intent |
  |Document Retriever       |Retrieve relevant knowledge |
  |Architecture Agent       |Generate architecture proposals |
  |Business Analyst Agent   |Generate requirements and user stories |
  |SQL Agent                |Generate SQL from database context |
  |Retrieval Evaluator      |Check retrieved context relevance | 
  |Response Evaluator       |Check answer quality and completeness |

### Important LangGraph concepts to implement

-   State management
-   Conditional edges
-   Nodes and transitions
-   Cyclic workflows
-   Retry mechanisms
-   Human-in-the-loop approval
-   Checkpointing and persistence

For example, if retrieval quality is poor, the graph should rewrite the
query and retrieve again rather than immediately generating an answer.

------------------------------------------------------------------------

## 5. RAG implementation -- Advanced capabilities

Do not stop at basic vector similarity search. Implement the following
progressively.

  Feature                  What you will learn
  ------------------------ ---------------------------------------------
  Basic RAG                Chunking, embeddings, retrieval
  Metadata filtering       Filter by project, document type, version
  Hybrid search            Combine keyword and vector search
  Reranking                Improve retrieved document relevance
  Query rewriting          Improve ambiguous questions
  Multi-query retrieval    Search using multiple interpretations
  Parent-child retrieval   Retrieve smaller chunks with larger context
  Contextual compression   Remove irrelevant retrieved information
  Citation generation      Reference original document and page
  RAG evaluation           Measure retrieval and answer quality

### Vector database comparison

  -----------------------------------------------------------------------
  Vector database                     Strengths / use cases
  ----------------------------------- -----------------------------------
  Qdrant                              Vector search, metadata filtering,
                                      hybrid retrieval; recommended
                                      starting point

  Azure AI Search                     Enterprise search, hybrid retrieval
                                      and Azure integration

  Chroma                              Simple local experimentation and
                                      prototyping

  Pinecone                            Managed vector database service
  -----------------------------------------------------------------------

**Recommendation:** Start with Qdrant locally using Docker, then
implement Azure AI Search as an alternative backend. This gives you
practical experience comparing a dedicated vector database with a
managed enterprise search service.

------------------------------------------------------------------------

## 6. LangFuse -- Observability

Use LangFuse to understand what happens internally when a user asks a
question.

### What to monitor

  Metric              What to monitor
  ------------------- -------------------------------------------
  Trace               Complete user request execution
  Span                Individual retrieval and processing steps
  Generation          LLM inputs and outputs
  Latency             Time taken by each component
  Token usage         Prompt and completion tokens
  Cost                Estimated model usage cost
  Retrieval quality   Relevance of retrieved documents
  Answer quality      Correctness and completeness
  Errors              Failed retrievals and LLM failures

### Example trace

``` text
User Question
    |
    +-- Question Classification
    |
    +-- Query Embedding
    |
    +-- Qdrant Retrieval
    |      |
    |      +-- Retrieved 5 chunks
    |
    +-- Reranking
    |
    +-- LLM Generation
    |
    +-- Answer Evaluation
    |
    +-- Final Response
```

You should be able to identify:

-   Why a particular answer was generated.
-   Which documents were retrieved.
-   Whether retrieval or generation caused a failure.
-   How many tokens were consumed.
-   Which workflow node took the most time.

### Additional challenge

Implement feedback scoring, such as thumbs-up/thumbs-down, and use the
feedback to improve retrieval and answer evaluation.

**Security note:** For enterprise usage, avoid recording sensitive
document content, credentials, or personal data in observability traces.
Configure masking and access controls.

------------------------------------------------------------------------

## 7. Implementation roadmap

A reasonable target is **4--6 weeks of part-time development**.

### Phase 1 -- Basic RAG

-   Set up Python and FastAPI.
-   Implement document ingestion.
-   Process PDF and DOCX files.
-   Implement chunking and embeddings.
-   Integrate Qdrant.
-   Build a basic question-answering chatbot.

**Deliverable:** Working document-based RAG chatbot.

### Phase 2 -- Advanced RAG

-   Implement hybrid search.
-   Add metadata filtering.
-   Implement query rewriting.
-   Add reranking.
-   Generate source citations.

**Deliverable:** More reliable retrieval pipeline.

### Phase 3 -- LangGraph

-   Implement question classification.
-   Add multiple specialised agents.
-   Build conditional workflows.
-   Add retrieval retry loops.
-   Implement human approval.

**Deliverable:** Multi-agent engineering assistant.

### Phase 4 -- LangFuse

-   Trace all workflows.
-   Capture retrieval and generation spans.
-   Monitor latency and token usage.
-   Add feedback and evaluation scores.

**Deliverable:** Observable AI application.

### Phase 5 -- Evaluation and security

-   Create a RAG evaluation dataset.
-   Measure retrieval precision and recall.
-   Evaluate faithfulness and answer relevance.
-   Implement document-level access controls.
-   Add prompt injection protection.
-   Add SQL execution safeguards.

**Deliverable:** Enterprise-ready prototype.

### Phase 6 -- Deployment

-   Containerise services using Docker Compose.
-   Build the React frontend.
-   Implement authentication.
-   Set up a CI/CD pipeline.
-   Deploy to Azure.

**Deliverable:** Deployable end-to-end application.

------------------------------------------------------------------------

## 8. Suggested GitHub repository structure

``` text
engineering-knowledge-copilot/
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── core/
│   │   ├── ingestion/
│   │   ├── embeddings/
│   │   ├── vectorstore/
│   │   ├── retrieval/
│   │   ├── agents/
│   │   ├── graphs/
│   │   ├── evaluation/
│   │   └── observability/
│   └── main.py
│
├── frontend/
│   └── React application
│
├── data/
│   ├── sample_documents/
│   └── evaluation_dataset/
│
├── infrastructure/
│   ├── docker-compose.yml
│   └── qdrant/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── evaluation/
│
├── notebooks/
│   ├── embeddings.ipynb
│   ├── vector_search.ipynb
│   └── rag_evaluation.ipynb
│
├── docs/
│   ├── architecture.md
│   ├── rag-design.md
│   └── security.md
│
└── README.md
```

------------------------------------------------------------------------

## 9. Final project acceptance criteria

  Capability   Acceptance criteria
  ------------ ------------------------------------------------------
  RAG          Answers questions using uploaded documents
  Retrieval    Returns relevant top-5 chunks
  Citations    Provides source document and page
  LangGraph    Executes conditional multi-agent workflows
  LangChain    Handles ingestion, retrieval and LLM integration
  Qdrant       Stores and searches document embeddings
  LangFuse     Displays complete traces and token usage
  Evaluation   Measures retrieval relevance and answer faithfulness
  Security     Prevents unauthorised document access
  Deployment   Runs using Docker Compose

------------------------------------------------------------------------

## 10. Recommended learning resources

Follow the official documentation while implementing the project:

-   [LangChain
    documentation](https://python.langchain.com/docs/introduction/)
-   [LangGraph
    documentation](https://docs.langchain.com/oss/python/langgraph/overview)
-   [Qdrant documentation](https://qdrant.tech/documentation/)
-   [LangFuse documentation](https://langfuse.com/docs)
-   [FastAPI documentation](https://fastapi.tiangolo.com/)

------------------------------------------------------------------------

## 11. Recommended implementation order

Do not start with multiple agents immediately.

1.  Build a working RAG pipeline with LangChain and Qdrant.
2.  Understand embeddings, chunking, retrieval and reranking.
3.  Introduce LangGraph to orchestrate workflows.
4.  Add LangFuse to observe each component.
5.  Add evaluation, security and deployment.

This sequence helps establish the fundamentals before introducing more
complex agent orchestration.

## 12. Suggested future enhancements

Once the core project works, consider adding:

-   GitHub repository ingestion and code-aware retrieval.
-   Integration with Azure AI Search.
-   Role-based access control and tenant isolation.
-   Document versioning and incremental indexing.
-   Human approval for SQL execution and other consequential actions.
-   Automated evaluation pipelines in CI/CD.
-   Comparison of different embedding models and LLMs.
-   Cost and latency optimisation.
-   A pre-sales workflow that converts customer requirements into PRDs,
    architecture proposals, estimates and proof-of-concept plans.

## Expected outcome

At completion, you will have a practical portfolio project
demonstrating:

-   Retrieval-Augmented Generation (RAG)
-   Vector databases and semantic search
-   LangChain application development
-   LangGraph agent orchestration
-   LLM observability using LangFuse
-   RAG evaluation and quality measurement
-   Enterprise AI security
-   Full-stack development and cloud deployment

The result is more than a chatbot: it is an observable, evaluated and
extensible AI engineering assistant.
