# CompliCS — Corporate Compliance RAG Assistant (India)

This project implements a **Retrieval-Augmented Generation (RAG)** system focused on **Indian corporate compliance**, specifically the **Companies Act, 2013**, the **Companies Rules, 2014** (Incorporation, Accounts, Management & Administration, Appointment & Qualification of Directors), and the **Limited Liability Partnership (LLP) Act, 2008** and **LLP Rules, 2009**.

The system assists in answering compliance-related queries by retrieving relevant statutory provisions, forms, and filing requirements from a curated document corpus and generating grounded, context-aware responses using a locally-hosted language model.

---

## Problem Statement

Indian corporate compliance involves navigating extensive legal texts, frequent amendments, and strict filing timelines. Professionals and students often need quick, accurate answers to questions such as:
- Applicable forms for a specific compliance event
- Due dates and penalties
- Relevant sections of the Companies Act, its Rules, or the LLP Act


---

## Solution Overview

This project uses a **RAG architecture** to:
1. Parse statutory PDFs into legally-structured chunks (Section/Rule → sub-section → proviso/explanation) and embed them into a vector database
2. Retrieve the most relevant legal context for a user query using **hybrid search** (BM25 keyword search + vector similarity, fused and reranked)
3. Generate responses strictly grounded in the retrieved content

This approach helps **reduce hallucinations** and improves reliability in the legal domain.

---

## Key Features

- Query-based retrieval from the Companies Act, Companies Rules, and LLP Act/Rules
- Deterministic, rule-based ingestion — statutory structure (Sections, Rules, sub-sections, clauses, provisos, explanations) is parsed with regex/state-machine logic rather than inferred by an LLM, so chunk boundaries and citation metadata are consistent and reproducible
- Hybrid retrieval (BM25 + embedding similarity via Reciprocal Rank Fusion) followed by cross-encoder reranking, since legal text has exact-match-critical terms (section numbers, defined terms) that pure semantic search often misses
- Context-aware, explainable responses with accurate source citations (Act, Section/Rule, sub-section, page)
- Conversational memory per session
- React (Vite) chat frontend, with a Streamlit UI kept as a lightweight alternative

---

## Tech Stack

- **Language:** Python
- **LLM:** Local, open-source LLM served via **Ollama** (Mistral)
- **Embeddings:** Local, via **Ollama** (`nomic-embed-text`)
- **Vector Database:** ChromaDB
- **Keyword Search:** BM25 (`rank-bm25`), fused with vector results via Reciprocal Rank Fusion
- **Reranking:** FlashRank (local cross-encoder)
- **RAG Framework:** LangChain
- **Backend API:** FastAPI
- **Frontend:** React (Vite); Streamlit kept as an alternate UI

---

## Data Sources

- Companies Act, 2013 (as amended up to 01.04.2021)
- Companies (Incorporation) Rules, 2014
- Companies (Accounts) Rules, 2014
- Companies (Management and Administration) Rules, 2014
- Companies (Appointment and Qualification of Directors) Rules, 2014
- Limited Liability Partnership Act, 2008
- LLP Rules, 2009
- Statutory forms (DIR-3 KYC, INC-20A, INC-22, MGT-7, LLP Form 8, LLP Form 11) and select annexures/circulars

All data used is publicly available (MCA gazette publications and forms).

**Known limitation:** these are bilingual (Hindi + English) gazette-style PDFs. Hindi/Devanagari-heavy pages are detected and dropped before parsing — only the English text is parsed structurally, and Hindi content is not translated or otherwise recovered. A small number of pages in at least one Rules PDF also have PDF font/encoding corruption (garbled glyph references) that is heuristically filtered out; this is a targeted fix for the one corruption pattern observed in this corpus, not a general PDF-repair guarantee.

---

## System Architecture

```
Source PDFs (corpus_raw_v1/)
        │
        ▼
PDF text extraction + Hindi/glyph-corruption page filtering
        │
        ▼
Deterministic legal-structure parser
  (Section/Rule → sub-section → clause → proviso/explanation,
   with hierarchical metadata; forms fall back to a plain splitter)
        │
        ▼
Embedding (Ollama nomic-embed-text) → ChromaDB
        │
        ▼
User Query
        │
        ▼
Hybrid Retrieval (BM25 + vector search, fused via RRF)
        │
        ▼
Reranking (FlashRank)
        │
        ▼
Context Injection → LLM Response Generation (Ollama Mistral)
        │
        ▼
Grounded answer + source citations
```

---

## Running the Project

Requires [Ollama](https://ollama.com) running locally with the `mistral` and `nomic-embed-text` models pulled.

```bash
# 1. Install backend dependencies
cd backend
pip install -r requirements.txt

# 2. Build the vector store from the source PDFs
python populate_database.py --reset --target prod   # first-time setup: writes to backend/chroma
# For re-ingestion on top of an existing production store, build into
# backend/chroma_v2 first (--target v2, the default) and spot-check it
# with scripts/inspect_retrieval.py / scripts/compare_stores.py before
# swapping it in for backend/chroma.

# 3. Start the API server
python server.py       # serves on http://localhost:8000

# 4. (Alternative UI) Streamlit
streamlit run app.py
```

```bash
# 5. Start the React frontend
cd frontend
npm install
npm run dev
```

`backend/scripts/inspect_retrieval.py` and `backend/scripts/compare_stores.py` are available for spot-checking chunk quality and comparing generated answers across store versions.

---

## Intended Use

- Academic project on RAG and vector databases
- Educational assistance for corporate law concepts
- Demonstration of legal-domain AI applications

 *This tool is for educational purposes only and does not constitute legal advice.*

---

##  Future Enhancements

- Annual compliance calendar generation
- Document upload and analysis
- Context expansion using parent-section metadata already captured during parsing


---

##  Author

**Vishesh Khadaria**  
**Dhruv Chaturvedi**
