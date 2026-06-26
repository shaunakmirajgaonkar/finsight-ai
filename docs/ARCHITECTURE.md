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
