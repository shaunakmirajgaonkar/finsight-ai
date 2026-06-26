# FinSight AI 📊

> 100% local financial document intelligence platform — no cloud, no API costs, works offline.

Upload annual reports, 10-K filings, and earnings transcripts. Ask questions in natural language. Get AI-powered insights, sentiment analysis, investment signals, and exportable PDF reports.

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![Ollama](https://img.shields.io/badge/LLM-Ollama%20Local-orange)
![FastAPI](https://img.shields.io/badge/Backend-FastAPI-009688)
![Streamlit](https://img.shields.io/badge/Frontend-Streamlit-FF4B4B)
![ChromaDB](https://img.shields.io/badge/VectorDB-ChromaDB-blueviolet)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## What is FinSight AI?

FinSight AI is a **RAG (Retrieval-Augmented Generation)** platform built for financial document analysis. It lets you upload any financial PDF, ask natural language questions, and receive accurate, source-cited answers — all running locally on your machine with zero internet dependency.

---

## Demo

```
Upload PDF → Select Document → Ask Question → Get Answer + Signal → Export PDF
```

**Example questions you can ask:**
```
What was total revenue and YoY growth?
What are the biggest risk factors?
What is management's outlook and guidance?
Should I invest in this company?
What drove gross margin changes this year?
```

---

## Features

| Feature | Description |
|---|---|
| 💬 **Natural Language Q&A** | Ask anything about your financial documents |
| 🔍 **Semantic Search** | Vector similarity search over document chunks |
| 📊 **Sentiment Analysis** | Bullish / Neutral / Bearish document scoring |
| 📈 **Investment Signals** | BUY / HOLD / SELL with confidence percentage |
| 📝 **Executive Summary** | AI-generated 4-paragraph document summary |
| ⚖️ **Multi-doc Compare** | Compare two documents on any aspect |
| 📄 **PDF Export** | Full conversation export with sources and signals |
| 🔒 **100% Local** | Zero cloud, zero API costs, works fully offline |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Streamlit UI                          │
│         Chat · Summary · Compare · Documents            │
└──────────────────────┬──────────────────────────────────┘
                       │ HTTP REST
┌──────────────────────▼──────────────────────────────────┐
│                   FastAPI Backend                        │
│    /upload · /chat · /summary · /compare                │
└──────┬───────────────┬──────────────────┬───────────────┘
       │               │                  │
┌──────▼──────┐ ┌──────▼──────┐ ┌────────▼────────┐
│   PyMuPDF   │ │  ChromaDB   │ │  Ollama LLM     │
│  pdfplumber │ │  MiniLM-L6  │ │  llama3.2:3b    │
│  DOCX parse │ │  embeddings │ │  local inference│
└─────────────┘ └─────────────┘ └─────────────────┘
```

### Data Flow

**Upload:**
```
PDF/DOCX → Parse pages → Chunk text → Embed (MiniLM) → Store (ChromaDB) → Sentiment score
```

**Query:**
```
Question → Embed → Semantic search → Top-6 chunks → Ollama LLM → Answer + Signal
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **LLM** | Ollama + Llama 3.2 3B (runs on CPU) |
| **Embeddings** | sentence-transformers `all-MiniLM-L6-v2` (22MB, offline) |
| **Vector Store** | ChromaDB (local persistent, no Docker needed) |
| **Backend** | FastAPI + Uvicorn |
| **Frontend** | Streamlit |
| **PDF Parsing** | PyMuPDF + pdfplumber |
| **DOCX Parsing** | python-docx |
| **Text Splitting** | LangChain RecursiveCharacterTextSplitter |
| **NLP** | VADER Sentiment + financial lexicon overrides |
| **PDF Export** | ReportLab |
| **Data Models** | Pydantic v2 |
| **Settings** | pydantic-settings + .env |

---

## Hardware Requirements

| RAM | Recommended Model | Notes |
|---|---|---|
| 4 GB | `llama3.2:3b` | Fast on CPU, good for testing |
| 8 GB | `llama3.1:8b` | Better quality answers |
| 16 GB+ | `llama3.1:70b` | Best quality, needs GPU |

---

## Installation

### Step 1 — Install Ollama

```bash
# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows — download from https://ollama.com/download
```

### Step 2 — Pull a model

```bash
ollama pull llama3.2:3b
```

### Step 3 — Clone and setup

```bash
git clone https://github.com/shaunakmirajgaonkar/finsight-ai.git
cd finsight-ai

python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt

cp .env.example .env
```

### Step 4 — Run

```bash
# Terminal 1 — Ollama (AI engine)
ollama serve

# Terminal 2 — FastAPI backend
uvicorn app.main:app --reload --port 8000

# Terminal 3 — Streamlit frontend
streamlit run streamlit_app.py
```

Open **http://localhost:8501** in your browser.

---

## Usage

### 1. Upload a document
- Click the upload box in the sidebar
- Select any financial PDF or DOCX
- Enter company name, fiscal year, document type
- Click **Process Document**
- Wait 30–60 seconds for parsing and embedding

### 2. Ask questions
- Tick the document checkbox in the sidebar
- Type any question in the chat input
- Or click a suggestion chip
- Get answer with source page citations and investment signal

### 3. Generate summary
- Go to **Summary** tab
- Click **Generate Executive Summary**
- Get a 4-paragraph AI summary with sentiment badge

### 4. Compare documents
- Upload and select 2+ documents
- Go to **Compare** tab
- Enter what to compare
- Click **Run Comparison**

### 5. Export PDF report
- After asking questions, scroll to bottom of Chat tab
- Click **Generate PDF**
- Download the formatted report with all Q&A, sources, and signals

---

## Project Structure

```
finsight-ai/
├── app/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── chat.py              # /chat /summary /compare endpoints
│   │   └── documents.py         # /upload /list /delete endpoints
│   ├── core/
│   │   ├── __init__.py
│   │   └── config.py            # Pydantic settings from .env
│   ├── models/
│   │   ├── __init__.py
│   │   └── schemas.py           # Request/Response Pydantic models
│   ├── services/
│   │   ├── __init__.py
│   │   ├── claude_service.py    # Ollama LLM — Q&A, KPI, signals, summary
│   │   ├── document_manager.py  # Upload pipeline orchestrator
│   │   ├── parser.py            # PDF + DOCX text extraction
│   │   ├── sentiment.py         # VADER + financial lexicon
│   │   └── vectorstore.py       # ChromaDB + MiniLM embeddings
│   ├── __init__.py
│   └── main.py                  # FastAPI app entry point
├── data/
│   ├── uploads/                 # Uploaded documents (gitignored)
│   └── vectorstore/             # ChromaDB files (gitignored)
├── docs/
│   ├── ARCHITECTURE.md          # Detailed architecture docs
│   └── API.md                   # Full API reference
├── tests/
│   └── test_pipeline.py         # End-to-end integration tests
├── streamlit_app.py             # Complete Streamlit UI + PDF export
├── requirements.txt
├── .env.example
├── .gitignore
├── CHANGELOG.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── SECURITY.md
├── ACKNOWLEDGMENTS.md
└── README.md
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `POST` | `/api/documents/upload` | Upload and process PDF/DOCX |
| `GET` | `/api/documents/` | List all documents |
| `DELETE` | `/api/documents/{id}` | Delete a document |
| `POST` | `/api/chat` | Ask a natural language question |
| `POST` | `/api/summary` | Generate executive summary |
| `POST` | `/api/compare` | Compare two documents |

Interactive Swagger UI: **http://localhost:8000/docs**

---

## Configuration

Edit `.env` to configure:

```env
# LLM model (change to llama3.1:8b for better quality)
OLLAMA_MODEL=llama3.2:3b

# Embedding model
EMBEDDING_MODEL=all-MiniLM-L6-v2

# RAG settings
CHUNK_SIZE=1000
CHUNK_OVERLAP=150
TOP_K_CHUNKS=6
```

---

## Supported Document Types

| Type | Extension | Notes |
|---|---|---|
| Annual Report | `.pdf` | Full text extraction |
| 10-K Filing | `.pdf` | Full text extraction |
| Earnings Transcript | `.pdf` | Full text extraction |
| Word Document | `.docx` | Paragraph extraction |
| Other | `.doc` | Basic support |

Max file size: 50MB

---

## Why 100% Local?

| Concern | How FinSight handles it |
|---|---|
| **Privacy** | Documents never leave your machine |
| **Cost** | Zero API costs, runs free forever |
| **Speed** | No network latency for embeddings |
| **Offline** | Works without internet after setup |
| **Control** | You own the model and data |

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Backend offline` | Run `uvicorn app.main:app --reload --port 8000` |
| `Connection refused on 11434` | Run `ollama serve` |
| Upload hangs forever | Run `ollama pull llama3.2:3b` first |
| `ModuleNotFoundError` | Run `source venv/bin/activate` |
| Port already in use | Run `lsof -ti:8000 \| xargs kill -9` |

---

## Built By

**Shaunak Mirajgaonkar**
BE Computer Engineering · MMCOE Pune (SPPU) · 2025
Internship Project @ Healthnexaa

[![GitHub](https://img.shields.io/badge/GitHub-shaunakmirajgaonkar-black?logo=github)](https://github.com/shaunakmirajgaonkar)

---

## License

MIT License — free to use, modify, and distribute.
See [LICENSE](LICENSE) for full details.
