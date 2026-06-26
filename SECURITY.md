# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| 1.0.x | ✅ Active |

---

## Reporting a Vulnerability

**Please do NOT open a public GitHub issue for security vulnerabilities.**

If you discover a security vulnerability, report it privately:

📧 **Email:** shaunakmirajgaonkar@gmail.com
**Subject:** `[SECURITY] FinSight AI — <brief description>`

### What to include

- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

### Response timeline

| Action | Timeline |
|---|---|
| Acknowledgement | Within 24 hours |
| Initial assessment | Within 48 hours |
| Fix or mitigation | Within 7 days |
| Public disclosure | After fix is released |

---

## Security Architecture

### Why FinSight AI is inherently secure

| Concern | How it is handled |
|---|---|
| **Data privacy** | All documents stay on your local machine |
| **No external calls** | Zero data sent to any server or API |
| **No user accounts** | No login, no stored credentials |
| **No database server** | ChromaDB runs as a local file store |
| **LLM inference** | Runs via Ollama on localhost only |
| **Embeddings** | Computed locally via sentence-transformers |

---

## Security Best Practices

### For users

- Never commit your `.env` file — it is gitignored by default
- Do not expose port 8000 or 8501 to the public internet
- Keep Ollama updated — `ollama pull llama3.2:3b` to refresh
- Use a virtual environment — never install globally
- Regularly update dependencies — `pip install -r requirements.txt --upgrade`

### For contributors

- Never hardcode API keys, tokens, or passwords
- Always use environment variables via `config.py`
- Sanitize all user inputs before passing to LLM
- Do not log sensitive document content
- Validate file types and sizes on upload

---

## Known Limitations

- This project is designed for **local single-user use only**
- Not hardened for production multi-user deployment
- No authentication or authorization layer
- CORS is set to `allow_origins=["*"]` — restrict before exposing publicly

---

## Dependency Security

To check for known vulnerabilities in dependencies:

```bash
pip install pip-audit
pip-audit
```

---

## Contact

**Shaunak Mirajgaonkar**
📧 shaunakmirajgaonkar@gmail.com
🐙 [@shaunakmirajgaonkar](https://github.com/shaunakmirajgaonkar)
