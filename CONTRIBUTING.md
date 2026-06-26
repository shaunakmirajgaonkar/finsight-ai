# Contributing to FinSight AI

Thank you for your interest in contributing to FinSight AI!
Every contribution is welcome — bug fixes, features, docs, or tests.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Reporting Bugs](#reporting-bugs)
- [Suggesting Features](#suggesting-features)
- [Submitting Code](#submitting-code)
- [Commit Message Format](#commit-message-format)
- [Code Style](#code-style)
- [Project Structure](#project-structure)

---

## Code of Conduct

This project follows our [Code of Conduct](CODE_OF_CONDUCT.md).
By participating, you agree to uphold it.

---

## Getting Started

```bash
# 1. Fork the repository on GitHub

# 2. Clone your fork
git clone https://github.com/YOUR_USERNAME/finsight-ai.git
cd finsight-ai

# 3. Set upstream remote
git remote add upstream https://github.com/shaunakmirajgaonkar/finsight-ai.git

# 4. Create virtual environment
python3 -m venv venv
source venv/bin/activate

# 5. Install dependencies
pip install -r requirements.txt

# 6. Configure environment
cp .env.example .env

# 7. Pull Ollama model
ollama pull llama3.2:3b

# 8. Run the project
ollama serve                                          # Terminal 1
uvicorn app.main:app --reload --port 8000            # Terminal 2
streamlit run streamlit_app.py                        # Terminal 3
```

---

## How to Contribute

### Reporting Bugs

1. Search [existing issues](https://github.com/shaunakmirajgaonkar/finsight-ai/issues) first
2. If not found, open a new issue with:
   - Clear descriptive title
   - Steps to reproduce
   - Expected vs actual behaviour
   - Your OS, Python version, Ollama version
   - Error message or screenshot if available

**Bug report template:**
