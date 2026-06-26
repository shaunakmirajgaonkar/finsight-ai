# Architecture

## System Overview
┌─────────────────────────────────────────────────────────┐

│                    Streamlit UI                          │

│         Chat · Summary · Compare · Documents            │

│                   localhost:8501                        │

└──────────────────────┬──────────────────────────────────┘

                │ HTTP REST

┌──────────────────────▼──────────────────────────────────┐

│                   FastAPI Backend                        │

│    /upload · /chat · /summary · /compare                │

│                   localhost:8000                        │

└──────┬───────────────┬──────────────────┬───────────────┘

│               │                  │

┌──────▼──────┐ ┌──────▼──────┐ ┌────────▼────────┐

│   Document  │ │  ChromaDB   │ │  Ollama LLM     │

│   Parsers   │ │  Vector DB  │ │  localhost:11434 │

│             │ │             │ │                 │

│  PyMuPDF    │ │  MiniLM-L6  │ │  llama3.2:3b    │

│  pdfplumber │ │  embeddings │ │  local inference│

│  python-docx│ │  cosine sim │ │  0 API cost     │

└─────────────┘ └─────────────┘ └─────────────────┘

   │               │

┌──────▼───────────────▼──────────────────────────────────┐

│                  Local File System                       │

│   data/uploads/     — uploaded PDFs                     │

│   data/vectorstore/ — ChromaDB persistent files         │

│   data/uploads/_metadata.json — document metadata       │

└─────────────────────────────────────────────────────────┘
---

## Component Breakdown

### 1. Streamlit UI (`streamlit_app.py`)

The frontend is a single-file Streamlit application with 4 tabs:

| Tab | Purpose |
|---|---|
| Chat | Natural language Q&A with source citations and signals |
| Summary | AI-generated executive summary of a document |
| Compare | Side-by-side comparison of two documents |
| Documents | View, select, and delete uploaded documents |

Key design decisions:
- `st.form` used for chat input to prevent re-run loops
- Session state manages chat history, selected docs, documents list
- No `st.rerun()` after API calls — Streamlit re-renders naturally after form submit
- PDF export uses ReportLab with no temporary files — pure in-memory bytes

---

### 2. FastAPI Backend (`app/`)

RESTful API with 6 endpoints across 2 routers:
app/

├── api/

│   ├── documents.py   → /api/documents/* (upload, list, delete)

│   └── chat.py        → /api/chat, /api/summary, /api/compare

├── core/

│   └── config.py      → Pydantic settings from .env

├── models/

│   └── schemas.py     → Request/Response models

├── services/

│   ├── parser.py          → PDF + DOCX extraction

│   ├── vectorstore.py     → ChromaDB operations

│   ├── sentiment.py       → VADER scoring

│   ├── claude_service.py  → Ollama LLM calls

│   └── document_manager.py → Upload pipeline orchestrator

└── main.py            → FastAPI app with CORS
---

### 3. Document Ingestion Pipeline
User uploads file

│

▼

Save to data/uploads/{uuid}_{filename}

│

▼

Parse (PyMuPDF → pdfplumber fallback)

│

▼

Extract pages → list of {page: int, text: str}

│

▼

LangChain RecursiveCharacterTextSplitter

chunk_size=1000, chunk_overlap=150

│

▼

sentence-transformers MiniLM-L6-v2

384-dimensional vectors

│

▼

ChromaDB upsert (batches of 500)

│

▼

VADER sentiment scoring (first 50 pages)

│

▼

Ollama KPI extraction (revenue, EPS, margins)

│

▼

Save metadata to _metadata.json
---

### 4. Query Pipeline
User question

│

▼

MiniLM-L6-v2 embeds question → 384-dim vector

│

▼

ChromaDB cosine similarity search

Filter by selected doc_ids

Return top-6 chunks

│

▼

Build context string from chunks

[Excerpt 1 — Page N]

text...
[Excerpt 2 — Page N]

text...

│

▼

Ollama API call (llama3.2:3b)

System: financial analyst persona

User: context + question

temperature=0.1 (deterministic)

│

▼

Parse answer text

│

▼

Generate BUY/HOLD/SELL signal (separate Ollama call)

│

▼

Return ChatResponse to Streamlit
---

### 5. Vector Store (`vectorstore.py`)

```python
# Embedding function — runs locally, no API
_embed_fn = SentenceTransformerEmbeddingFunction(
    model_name="all-MiniLM-L6-v2"  # 22MB, downloads once
)

# ChromaDB persistent client
_client = chromadb.PersistentClient(path="./data/vectorstore")

# Single collection for all documents
_collection = _client.get_or_create_collection(
    name="financial_docs",
    embedding_function=_embed_fn,
    metadata={"hnsw:space": "cosine"},  # cosine similarity
)
```

Chunk IDs follow the pattern: `{doc_id}_p{page}_c{chunk_index}`
This allows filtering by document, page, or chunk.

---

### 6. Sentiment Analysis (`sentiment.py`)

VADER with 20+ financial lexicon overrides:

```python
FINANCIAL_LEXICON = {
    # Bullish signals
    "outperform": +2.5,
    "exceeded":   +2.0,
    "raised guidance": +2.5,

    # Bearish signals
    "headwinds":  -1.5,
    "missed":     -2.0,
    "lowered guidance": -2.5,
    "impairment": -2.0,
}
```

Scores first 50 pages, weights first 20 sentences (recency bias).
Maps compound score → `bullish` (≥0.10) / `neutral` / `bearish` (≤-0.10)

---

### 7. LLM Service (`claude_service.py`)

Four Ollama calls, each with `temperature=0.1`:

| Function | Purpose | Max tokens |
|---|---|---|
| `answer_question()` | RAG Q&A | 1500 |
| `extract_kpis()` | JSON KPI extraction | 200 |
| `generate_recommendation()` | BUY/HOLD/SELL signal | 150 |
| `summarize_document()` | Executive summary | 600 |

All JSON responses use regex extraction as fallback
in case the model adds markdown fences.

---

## Key Design Decisions

| Decision | Reason |
|---|---|
| Local LLM (Ollama) | Zero cost, full privacy, works offline |
| ChromaDB | No Docker, persists to disk, simple Python API |
| MiniLM-L6-v2 | 22MB, fast CPU inference, good semantic accuracy |
| Single ChromaDB collection | Simpler, filtered by doc_id on query |
| JSON metadata store | No database setup, good for single-user local use |
| st.form for chat | Prevents Streamlit re-run loop on API calls |
| PyMuPDF + pdfplumber fallback | Handles both text and table-heavy PDFs |
| Pydantic v2 models | Type safety, automatic validation, clean serialization |

---

## Performance

| Operation | Typical time |
|---|---|
| PDF parsing (100 pages) | 2-5 seconds |
| Embedding (100 pages) | 10-30 seconds |
| Semantic search | < 1 second |
| LLM answer generation | 10-30 seconds (CPU) |
| Sentiment analysis | < 1 second |
| PDF export | 1-3 seconds |

---

## Scaling Limitations

This architecture is designed for single-user local use.
For multi-user production deployment you would need:

- PostgreSQL instead of JSON metadata store
- Celery + Redis for async document processing
- Pinecone or Weaviate instead of local ChromaDB
- JWT authentication
- Gunicorn + Nginx instead of Uvicorn dev server
- Docker + docker-compose
- React frontend instead of Streamlit
- 
