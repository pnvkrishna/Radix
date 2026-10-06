# RAG Master Preparation Index

> **Goal:** Build strong foundations and practical expertise in **Retrieval-Augmented Generation** (RAG).
>
> This index is designed to help me:
>
> - Understand RAG deeply
> - Implement RAG systems
> - Debug RAG systems
> - Improve retrieval quality
> - Understand production architecture
> - Compare different RAG approaches
> - Explain concepts clearly
> - Teach RAG to others




**Core Learning Principle**


> > Don't learn RAG as a collection of LangChain APIs.
> > Learn RAG as a **retrieval engineering problem**.


* LangChain API నేర్చుకోవడం: "ఈ function ఎలా use చేయాలి?"

* RAG Engineering: "ఈ problem కి ఏ approach ఎందుకు use చేయాలి?"

# 00. How to Use This Handbook

For every class, follow:

```text
Watch
  ↓
Understand
  ↓
Implement
  ↓
Experiment
  ↓
Break
  ↓
Debug
  ↓
Explain
  ↓
Teach
  ↓
Document
  ↓
Revise
````

A topic is considered **mastered** only when I can:

* Explain it simply
* Explain why it exists
* Explain how it works internally
* Draw its architecture
* Implement it
* Modify the implementation
* Debug common failures
* Compare alternatives
* Explain production considerations
* Teach it to someone else

---

# 01. LLM Foundations

## RAG-01 — How LLMs Generate Text

**Date:** 23-Jun-2026

### Core Concepts

* Large Language Models
* Machine Learning → Deep Learning → Neural Networks
* Transformer architecture
* Tokens
* Tokenization
* Vocabulary
* Token IDs
* Next-token prediction
* Probability distribution
* Temperature
* End of Sequence (EOS)
* Prompt quality
* Model provider SDKs
* OpenAI
* Gemini
* Claude
* Why abstraction frameworks such as LangChain exist

### Master Question

> How does an LLM generate a response from a user's prompt?

---

## RAG-02 — How LLMs Understand Prompts

**Date:** 26-Jun-2026

### Core Concepts

* LLMs don't directly process human-readable text
* Tokenization
* Token IDs
* Embeddings
* Transformer input
* Representation of text
* Probability generation

### Master Question

> What happens to my prompt between typing it and receiving an answer?

---

## RAG-03 — Vectors & Semantic Search

**Date:** 27-Jun-2026

### Core Concepts

* Vector
* Dimensions
* Vector space
* Semantic meaning
* Semantic similarity
* Vector relationships
* Vector databases
* Why keyword matching is insufficient
* Semantic search

### Master Question

> How can a computer search by meaning instead of only matching words?

---

# 02. LangChain & LLM Application Foundations

## RAG-04 — LCEL & Chaining

**Date:** 29-Jun-2026

### Core Concepts

* Prompt
* LLM
* Chain
* LCEL
* `|`
* System Prompt
* User Prompt
* Messages
* SystemMessage
* HumanMessage
* AIMessage
* ToolMessage
* Chat models
* Output parsers
* JSON structured output
* LangChain
* Gemini integration
* Environment variables
* API keys

### Master Question

> How do I compose multiple LLM components into a reusable pipeline?

---

## RAG-05 — Runnable & RAG Fundamentals

**Date:** 30-Jun-2026

### Core Concepts

### Runnable

* Runnable interface
* `invoke`
* `ainvoke`
* `batch`
* `stream`

### RAG

* Retrieval-Augmented Generation
* Indexing phase
* Retrieval phase
* External/private data
* Vector databases
* Vector stores
* Document loaders
* Chunking
* Embeddings
* Retrievers

### Master Question

> What are the two fundamental phases of RAG?

```text
INDEXING
   ↓
Private Data
   ↓
Prepare for Retrieval


RETRIEVAL
   ↓
User Question
   ↓
Find Relevant Information
   ↓
LLM
```

---

## RAG-06 — GCP & Gemini Setup

**Date:** 02-Jul-2026

### Core Concepts

* Google Cloud
* GCP project
* Vertex AI
* Gemini
* Google Cloud authentication
* `gcloud`
* Application Default Credentials
* LangChain + Gemini
* Environment configuration
* `ChatPromptTemplate`
* Output parsers

### Master Question

> How do I connect a cloud-hosted Gemini model to a Python/LangChain application?

---

# 03. RAG Ingestion Pipeline

## RAG-07 — Document Loading

**Date:** 04-Jul-2026

### Core Concepts

* Document
* Document object
* DocumentLoader
* Text files
* PDF files
* CSV files
* Directory loading
* URLs
* `langchain-community`
* Source data
* Metadata

### Master Question

> How do I convert different data sources into a common document representation?

---

## RAG-08 — Chunking Strategies

**Date:** 06-Jul-2026

### Core Concepts

* Why chunking is required
* Chunk
* Document splitting
* Chunk size
* Chunk overlap
* Recursive splitting
* `RecursiveCharacterTextSplitter`
* Metadata during ingestion
* Ingestion pipeline

### Important Relationship

```text
Document
   ↓
Split
   ↓
Chunks
   ↓
Embedding
   ↓
Vector Store
```

### Master Question

> How does chunk size affect retrieval quality?

---

## RAG-09 — Embeddings & Vector Databases

**Date:** 07-Jul-2026

### Core Concepts

### Embeddings

* Embedding
* Meaning representation
* Word2Vec
* Embedding models
* Open-source embeddings
* Cloud embedding services
* Embedding dimensions
* Token limits
* `embed_query`
* `embed_documents`

### Vector Stores

* Vector store
* Vector database
* Chroma
* FAISS
* Weaviate
* Pinecone
* pgvector
* MongoDB
* Cloud vector databases

### Indexing Exercise

```text
City Documents
      ↓
RecursiveCharacterSplitter
      ↓
Chunks
      ↓
Embedding Model
      ↓
Chroma
      ↓
Metadata
      ↓
Vector Store
```

### Master Questions

> What is an embedding?

> Why do we need a vector database?

> How does semantic similarity work?

---

## RAG-10 — Building the First RAG System

**Date:** 08-Jul-2026

### Core Concepts

* First RAG application
* Indexing
* Retrieval
* Query
* Relevant chunks
* Context
* Prompt
* LLM
* Generated answer

### Fundamental Architecture

```text
DOCUMENTS
    ↓
LOAD
    ↓
CHUNK
    ↓
EMBED
    ↓
VECTOR STORE
    ↓
RETRIEVE
    ↓
CONTEXT
    ↓
PROMPT
    ↓
LLM
    ↓
ANSWER
```

### Master Question

> Can I build a complete basic RAG system without blindly copying a tutorial?

---

# 04. RAG with Different Data Types

## RAG-11 — RAG with Structured Data

**Date:** 09-Jul-2026

### Core Concepts

* Structured data
* Relational databases
* SQL
* Schema understanding
* Natural Language → Query
* SQLDatabaseToolkit
* MongoDB
* NoSQL
* Structured-data retrieval

### Mental Model

```text
User Question
      ↓
Understand Schema
      ↓
Natural Language
      ↓
Query
      ↓
Database
      ↓
Result
      ↓
LLM
      ↓
Answer
```

### Master Question

> How is RAG over structured data different from RAG over documents?

---

## RAG-12 — RAG Productionization

**Date:** 11-Jul-2026

### Core Concepts

* Production RAG
* Chunking quality
* Vector store quality
* RAG scoring
* Evaluation
* Guardrails
* Data variety
* Text
* Images
* PDF extraction
* Noise filtering
* Tables
* Flowcharts
* Document structure

### Real-World Case Study

**NCERT Teacher RAG**

```text
NCERT Mathematics Book
        ↓
Chapter
        ↓
Sections
        ↓
Text
        ↓
Images
        ↓
Tables
        ↓
Flowcharts
        ↓
RAG
```

### Master Question

> What changes when a RAG prototype becomes a production system?

---

## RAG-13 — Multimodal RAG & Image Captioning

**Date:** 13-Jul-2026

### Core Concepts

* Multimodal data
* Images in RAG
* Image meaning
* Image embeddings
* Multimodal RAG
* Image captioning
* Capturing image meaning
* Visual information retrieval

### Two Broad Approaches

```text
IMAGE
  │
  ├── Capture Meaning
  │      ↓
  │   Caption / Description
  │
  └── Embed Image
         ↓
      Vector Search
```

### Master Question

> How can a RAG system retrieve information contained inside images?

---

# 05. Advanced Chunking

## RAG-14 — Section-Aware Chunking

**Date:** 14-Jul-2026

**Status:** Partial / Class dropped in the middle

### Core Concepts

* Section-aware chunking
* Chapter structure
* Section boundaries
* Multi-page sections
* Document structure

### Master Question

> Why can page-based chunking fail when document meaning spans multiple pages?

---

## RAG-15 — Section-Aware Chunking

**Date:** 15-Jul-2026

### Core Concepts

* NCERT chapter structure
* Section detection
* Regex patterns
* Section boundaries
* Recursive splitting
* Chunk size
* Chunk overlap
* Maximum chunk size
* Combining page text

### Target Strategy

```text
Chapter
   ↓
Detect Sections
   ↓
Split by Section
   ↓
Split by Length
   ↓
Apply Overlap
   ↓
Final Chunks
```

### Example

```text
Section
   ↓
1000 character/token target
   ↓
150 overlap
```

### Master Question

> How can I preserve document structure while still enforcing a maximum chunk size?

---

## RAG-16 — Recursive Chunking & Vector Storage

**Date:** 16-Jul-2026

### Core Concepts

### Recursive Splitting

* Separators
* First separator
* Second separator
* Recursive splitting
* Chunk size
* Chunk boundaries

### Vector Storage

* Vector storage layer
* Vector dimensions
* Dense float arrays
* Unique IDs
* Metadata
* Payload/raw text

### Indexing Algorithms

* Flat index
* IVF
* HNSW
* Approximate nearest-neighbor search

### Distance Metrics

* Cosine similarity
* Dot product
* Euclidean distance

### Vector Database Operations

* Metadata filtering
* CRUD
* Updates
* Index creation

### Master Questions

> How does a vector database actually search millions of vectors?

> What is the difference between Flat, IVF and HNSW?

> How does a vector database measure similarity?

---

# 06. Advanced Retrieval

## RAG-17 — Dense, Sparse & Hybrid Retrieval

**Date:** 21-Jul-2026

### Dense Retrieval

Focus:

> Meaning

Uses:

* Embeddings
* Semantic similarity

---

### Sparse Retrieval

Focus:

> Keywords

Example:

* BM25
* Term matching

---

### Hybrid Retrieval

Combines:

```text
Dense Retrieval
      +
Sparse Retrieval
      ↓
Fusion
      ↓
Weighted Results
```

### Key Concepts

* Dense retrieval
* Sparse retrieval
* BM25
* Hybrid retrieval
* Fusion
* Ranking
* Weights
* Semantic search
* Keyword search

### Master Question

> When is semantic search not enough, and why does hybrid retrieval often perform better?

---

# 07. RAG Application Layer

## RAG-18 — RAG User Interface with Streamlit

**Date:** 22-Jul-2026

### Core Concepts

* RAG API
* Chat interface
* Streamlit
* Gradio
* Streamlit application
* Widgets
* Session state
* Callbacks
* Caching
* Cache data
* Cache resources

### Important Concept

Streamlit reruns the application when interaction occurs.

Therefore understand:

```text
Local Variable
      vs
Session State
```

### Master Question

> How do I turn a RAG backend into a usable application?

---

# 08. RAG Evaluation

## RAG-19 — RAG Evaluation with DeepEval

**Date:** 23-Jul-2026

### The Problem

LLM output is:

* Non-deterministic
* Variable
* Difficult to verify using exact string equality

### Evaluation Architecture

```text
Application
    ↓
Test Case
    ↓
Metric
    ↓
Evaluation Runner
    ↓
Quality Score
```

### DeepEval

Core concepts:

* TestCase
* Metric
* Evaluation Runner
* LLM evaluation
* Quality evaluation
* pytest integration

### Master Question

> How do I know whether my RAG system is actually good?

---

# 09. RAG Data Lifecycle

## RAG-20 — Document Updates & Incremental Indexing

**Date:** 25-Jul-2026

### The Problem

Source documents change.

Therefore:

```text
Source Data
     ↓
Changes
     ↓
Vector Index
     ↓
Must Update
```

### Strategies

#### Strategy 1 — Full Re-index

```text
Delete old index
      ↓
Process everything again
```

#### Strategy 2 — Content Hashing

```text
Document
   ↓
Hash
   ↓
Compare
   ↓
Only changed content
   ↓
Upsert
```

#### Strategy 3 — LangChain Indexing API

Use framework-supported indexing capabilities.

#### Strategy 4 — Document Versioning

```text
Version 1
Version 2
Version 3
   ↓
Soft Delete
```

### Master Question

> How do I keep a production RAG index synchronized with changing source documents?

---

# 10. The Complete RAG Knowledge Map

```text
                         RAG ENGINEERING
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
         LLM                 DATA              RETRIEVAL
          │                    │                    │
     ┌────┼────┐        ┌──────┼──────┐      ┌────┼────┐
     │    │    │        │      │      │      │    │    │
  Tokens Prompts Models  Text  Images DBs  Dense Sparse Hybrid
     │    │    │        │      │      │      │    │    │
     └────┴────┘        └──────┴──────┘      └────┴────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
                         RAG PIPELINE
                               │
                ┌──────────────┼──────────────┐
                ▼              ▼              ▼
             Ingestion     Retrieval       Generation
                │              │              │
             Loading        Search           LLM
             Chunking       Ranking          Prompt
             Embedding      Filtering        Context
             Indexing       Top-K             Answer
                │              │              │
                └──────────────┼──────────────┘
                               ▼
                     PRODUCTION RAG
                               │
           ┌───────────────────┼───────────────────┐
           ▼                   ▼                   ▼
        Evaluation             UI              Data Updates
           │                   │                   │
        DeepEval            Streamlit         Incremental
        Metrics             Session State      Indexing
        Testing             Caching            Versioning
```

---

# 11. RAG Quality Engineering

A strong RAG engineer should think about quality at every stage.

```text
Source Quality
      ↓
Extraction Quality
      ↓
Chunking Quality
      ↓
Embedding Quality
      ↓
Index Quality
      ↓
Retrieval Quality
      ↓
Context Quality
      ↓
Prompt Quality
      ↓
LLM Quality
      ↓
Answer Quality
```

### Key Principle

> **Bad retrieval cannot be completely fixed by a better prompt.**

Therefore:

```text
Better LLM
      ≠
Automatically Better RAG
```

---

# 12. Retrieval Engineering Checklist

When RAG gives a bad answer, investigate in this order:

```text
1. Was the source document loaded correctly?
2. Was the required information extracted?
3. Was noise removed?
4. Were sections preserved?
5. Were chunks created correctly?
6. Is chunk size appropriate?
7. Is overlap appropriate?
8. Is the embedding model suitable?
9. Are vectors stored correctly?
10. Is similarity search working?
11. Is metadata filtering correct?
12. Is Top-K appropriate?
13. Would sparse retrieval help?
14. Would hybrid retrieval help?
15. Is the retrieved context actually relevant?
16. Is the prompt using the retrieved context correctly?
17. Is the LLM generating faithfully?
18. Can evaluation metrics confirm the problem?
```

---

# 13. Practical Projects

## Project 1 — Basic Text RAG

```text
TXT / PDF
   ↓
Loader
   ↓
Chunking
   ↓
Embeddings
   ↓
Chroma
   ↓
Retriever
   ↓
Gemini
   ↓
Answer
```

---

## Project 2 — City Knowledge RAG

Requirements:

* Multiple city documents
* Recursive splitting
* Embeddings
* Chroma
* Metadata
* City filtering
* Semantic search

---

## Project 3 — NCERT Book RAG

Requirements:

* Chapter extraction
* Section detection
* Text extraction
* Image extraction
* Noise filtering
* Section-aware chunking
* Embeddings
* Vector storage
* Retrieval
* Multimodal handling

---

## Project 4 — Hybrid RAG

Implement:

```text
Dense Search
     +
BM25
     ↓
Fusion
     ↓
Ranked Results
     ↓
LLM
```

---

## Project 5 — Production RAG

Add:

* Streamlit UI
* Evaluation
* Metrics
* Metadata filtering
* Document updates
* Incremental indexing
* Versioning
* Error handling
* Logging

---

# 14. RAG Debugging Mindset

When something fails, don't immediately change code.

Ask:

```text
WHAT FAILED?
     ↓
WHERE DID IT FAIL?
     ↓
WHY DID IT FAIL?
     ↓
HOW CAN I PROVE IT?
     ↓
WHAT IS THE SMALLEST FIX?
     ↓
HOW DO I PREVENT IT AGAIN?
```

---

# 15. Teaching Mastery

For every concept, prepare these six explanations:

### Level 1 — One Sentence

Explain it to a complete beginner.

### Level 2 — Analogy

Explain it using a real-world example.

### Level 3 — Technical

Explain how it works.

### Level 4 — Code

Implement it in Python.

### Level 5 — Architecture

Show where it belongs in RAG.

### Level 6 — Production

Explain trade-offs, limitations, scaling and failure modes.

---

# 16. Interview Preparation Map

## LLM

* What is a token?
* How does next-token prediction work?
* What is temperature?
* What is a Transformer?

## Embeddings

* What is an embedding?
* Why do we need embeddings?
* What is embedding dimension?
* How do embedding models differ?

## Chunking

* Why chunk documents?
* What is chunk overlap?
* What is recursive splitting?
* What is section-aware chunking?
* How do you choose chunk size?

## Vector Databases

* What is a vector database?
* What is similarity search?
* Cosine vs dot product vs Euclidean?
* Flat vs IVF vs HNSW?
* What is metadata filtering?

## Retrieval

* Dense vs sparse retrieval?
* What is BM25?
* Why hybrid retrieval?
* What is Top-K?
* How do you improve retrieval quality?

## RAG

* What is RAG?
* Indexing vs retrieval?
* What happens during ingestion?
* What happens during query time?
* Why can RAG produce incorrect answers?

## Multimodal RAG

* How do you process images?
* Image captioning vs image embeddings?
* How do tables and diagrams affect RAG?

## Evaluation

* Why can't we use exact string matching?
* What is a test case?
* What is a metric?
* How do you evaluate retrieval quality?

## Production

* How do you update a RAG index?
* Full re-index vs incremental indexing?
* How do you handle deleted documents?
* How do you monitor RAG quality?
* How do you control cost?

---

# 17. RAG Master Revision Order

When revising the entire course, use this order:

```text
01. LLM Generation
        ↓
02. Prompt Processing
        ↓
03. Vectors
        ↓
04. Embeddings
        ↓
05. Documents
        ↓
06. Chunking
        ↓
07. Vector Stores
        ↓
08. Retrieval
        ↓
09. Basic RAG
        ↓
10. Structured RAG
        ↓
11. Multimodal RAG
        ↓
12. Advanced Chunking
        ↓
13. Dense Retrieval
        ↓
14. Sparse Retrieval
        ↓
15. Hybrid Retrieval
        ↓
16. RAG UI
        ↓
17. RAG Evaluation
        ↓
18. Document Updates
        ↓
19. Production RAG
```

---

# 18. Final Master Mental Model

Always remember:

```text
                 ┌─────────────────┐
                 │  SOURCE DATA    │
                 └────────┬────────┘
                          │
                          ▼
                    INGESTION
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
          Loading      Extraction    Cleaning
             │            │            │
             └────────────┼────────────┘
                          ▼
                       Chunking
                          │
                          ▼
                      Embeddings
                          │
                          ▼
                    Vector Storage
                          │
                          ▼
                       INDEXING
                          │
                          │
              ───── QUERY TIME ─────
                          │
                          ▼
                    User Question
                          │
                          ▼
                   Query Embedding
                          │
                          ▼
                      Retrieval
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           Dense        Sparse       Hybrid
             │            │            │
             └────────────┼────────────┘
                          ▼
                    Ranking / Filter
                          │
                          ▼
                    Relevant Context
                          │
                          ▼
                 Prompt + Context
                          │
                          ▼
                         LLM
                          │
                          ▼
                       Answer
                          │
                          ▼
                     EVALUATION
                          │
                          ▼
                   IMPROVEMENT LOOP
```

---

# 19. The Ultimate RAG Question

Everything in this handbook ultimately answers one question:

> **How can I reliably give an LLM the right information, from the right source, at the right time, in the right format, and measure whether the resulting answer is actually good?**

That is the core of RAG engineering.

---

# 20. Current Class Progress

| Phase                                | Classes         | Status |
| ------------------------------------ | --------------- | ------ |
| LLM Foundations                      | RAG-01 → RAG-03 | ✅      |
| LangChain Foundations                | RAG-04 → RAG-06 | ✅      |
| RAG Foundations                      | RAG-07 → RAG-10 | ✅      |
| Structured / Production / Multimodal | RAG-11 → RAG-13 | ✅      |
| Advanced Chunking                    | RAG-14 → RAG-16 | ✅      |
| Advanced Retrieval                   | RAG-17          | ✅      |
| RAG Application                      | RAG-18          | ✅      |
| RAG Evaluation                       | RAG-19          | ✅      |
| Document Lifecycle                   | RAG-20          | ✅      |

---

# 🎯 Final Goal

I am not trying to become someone who can simply:

```python
from langchain...
```

and copy a RAG tutorial.

I am becoming someone who can:

```text
UNDERSTAND
    ↓
DESIGN
    ↓
IMPLEMENT
    ↓
DEBUG
    ↓
EVALUATE
    ↓
OPTIMIZE
    ↓
PRODUCTIONIZE
    ↓
TEACH
```

**That is my RAG learning goal.**


