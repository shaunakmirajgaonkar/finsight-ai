# Changelog

All notable changes to FinSight AI are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.0] — 2025-06-26

### Added

#### Backend
- FastAPI REST backend with 6 endpoints
- Document upload pipeline — PDF and DOCX parsing
- PyMuPDF + pdfplumber for robust PDF text extraction
- python-docx for DOCX paragraph extraction
- LangChain RecursiveCharacterTextSplitter for intelligent chunking
- ChromaDB local vector store with persistent storage
- sentence-transformers MiniLM-L6-v2 for local embeddings
- Ollama integration for 100% local LLM inference
- Llama 3.2 3B support (runs on CPU, 4GB RAM)
- VADER sentiment analysis with custom financial lexicon
- BUY / HOLD / SELL investment signal generation
- KPI extraction from financial documents
- Executive summary generation
- Multi-document comparison endpoint
- Pydantic v2 data models and validation
- Environment-based configuration via .env

#### Frontend
- Streamlit UI with professional dark navy design
- Bloomberg terminal-inspired header bar
- Chat tab with natural language Q&A
- Suggestion chips for common financial questions
- Source citation expander with page numbers and relevance scores
- Investment signal badges with confidence percentage
- Summary tab with document metadata and executive summary
- Compare tab for multi-document analysis
- Documents tab with full metadata and management
- Sidebar with document upload, selection, and deletion
- Real-time document stats in terminal bar
- PDF export of full conversation with ReportLab
- Sentiment pills — Bullish / Neutral / Bearish

#### Infrastructure
- 100% local stack — zero cloud, zero API costs
- Works fully offline after initial model download
- Persistent ChromaDB vector store
- JSON-based document metadata store
- .env configuration system
- Comprehensive .gitignore

---

## [Unreleased]

### Planned
- PostgreSQL metadata store
- Streaming LLM responses
- Docker + docker-compose support
- User authentication
- Multiple LLM support (Gemini, OpenAI swap-in)
- Batch document upload
- Document tagging and search
- Chart and table extraction from PDFs
- Email PDF report feature
