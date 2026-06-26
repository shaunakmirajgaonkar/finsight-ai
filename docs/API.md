# API Reference

**Base URL:** `http://localhost:8000`
**Interactive Docs:** `http://localhost:8000/docs`
**Content-Type:** `application/json` (except file upload)

---

## Health Check

```http
GET /health
```

**Response**
```json
{
  "status": "ok",
  "service": "FinSight AI"
}
```

---

## Documents

### Upload Document

```http
POST /api/documents/upload
Content-Type: multipart/form-data
```

**Request body**

| Field | Type | Required | Description |
|---|---|---|---|
| `file` | File | ✅ | PDF or DOCX file (max 50MB) |
| `company` | string | ✅ | Company name e.g. `Apple Inc.` |
| `year` | integer | ✅ | Fiscal year e.g. `2024` |
| `doc_type` | string | ✅ | One of: `annual_report` `earnings_transcript` `10k` `10q` `other` |

**Response**
```json
{
  "doc_id": "550e8400-e29b-41d4-a716-446655440000",
  "message": "Processed 95 pages into 423 chunks.",
  "meta": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "filename": "apple_ar_2024.pdf",
    "company": "Apple Inc.",
    "year": 2024,
    "doc_type": "annual_report",
    "page_count": 95,
    "chunk_count": 423,
    "uploaded_at": "2025-06-26T10:30:00",
    "sentiment_score": 0.142,
    "sentiment_label": "bullish"
  }
}
```

---

### List Documents

```http
GET /api/documents/
```

**Response**
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "filename": "apple_ar_2024.pdf",
    "company": "Apple Inc.",
    "year": 2024,
    "doc_type": "annual_report",
    "page_count": 95,
    "chunk_count": 423,
    "uploaded_at": "2025-06-26T10:30:00",
    "sentiment_score": 0.142,
    "sentiment_label": "bullish"
  }
]
```

---

### Get Document

```http
GET /api/documents/{doc_id}
```

**Response** — Same as single object in list above.

---

### Delete Document

```http
DELETE /api/documents/{doc_id}
```

**Response**
```json
{
  "message": "Deleted 550e8400-e29b-41d4-a716-446655440000"
}
```

---

## Chat

### Ask a Question

```http
POST /api/chat
```

**Request body**
```json
{
  "question": "What was total revenue and YoY growth?",
  "doc_ids": ["550e8400-e29b-41d4-a716-446655440000"],
  "include_recommendation": true
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `question` | string | ✅ | Natural language question |
| `doc_ids` | array | ✅ | List of document IDs to query |
| `include_recommendation` | boolean | ❌ | Include BUY/HOLD/SELL signal (default: true) |

**Response**
```json
{
  "answer": "Total revenue was $391.0B for FY2024, representing a 2.0% increase year-over-year from $383.3B in FY2023...",
  "sources": [
    {
      "text": "Total net sales increased 2% or $7.8 billion during 2024 compared to 2023...",
      "page": 41,
      "doc_id": "550e8400-e29b-41d4-a716-446655440000",
      "relevance_score": 0.89
    },
    {
      "text": "The following table shows net sales by category...",
      "page": 42,
      "doc_id": "550e8400-e29b-41d4-a716-446655440000",
      "relevance_score": 0.81
    }
  ],
  "kpis": null,
  "recommendation": {
    "rating": "BUY",
    "confidence": 0.78,
    "rationale": "Strong services growth and record gross margins indicate healthy business fundamentals."
  },
  "sentiment": null
}
```

---

### Generate Executive Summary

```http
POST /api/summary
```

**Request body**
```json
{
  "doc_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response**
```json
{
  "doc_id": "550e8400-e29b-41d4-a716-446655440000",
  "company": "Apple Inc.",
  "year": 2024,
  "summary": "Apple Inc. delivered strong financial performance in FY2024...\n\nThe company's Services segment continued to be a key growth driver...\n\nKey risks include geopolitical tensions...\n\nManagement expressed confidence in the AI opportunity...",
  "sentiment_score": 0.142,
  "sentiment_label": "bullish"
}
```

---

### Compare Documents

```http
POST /api/compare
```

**Request body**
```json
{
  "doc_ids": [
    "550e8400-e29b-41d4-a716-446655440000",
    "660f9511-f30c-52e5-b827-557766551111"
  ],
  "aspect": "revenue growth and gross margins"
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `doc_ids` | array | ✅ | Minimum 2 document IDs |
| `aspect` | string | ❌ | What to compare (default: overall performance) |

**Response**
```json
{
  "comparison": "Comparing Apple Inc. 2024 and Microsoft 2024 on revenue growth and gross margins:\n\nApple reported total revenue of $391B (+2% YoY) with gross margin of 46.2%...\n\nMicrosoft reported revenue of $245B (+16% YoY) with gross margin of 69.8%...",
  "sources": [
    {
      "text": "excerpt from Apple doc...",
      "page": 41,
      "doc_id": "550e8400-e29b-41d4-a716-446655440000",
      "relevance_score": 0.87
    },
    {
      "text": "excerpt from Microsoft doc...",
      "page": 38,
      "doc_id": "660f9511-f30c-52e5-b827-557766551111",
      "relevance_score": 0.84
    }
  ]
}
```

---

## Error Responses

| Status | Meaning |
|---|---|
| `400` | Bad request — missing or invalid fields |
| `404` | Document not found |
| `413` | File too large (max 50MB) |
| `422` | No relevant content found for the question |
| `500` | Server error |

**Error format**
```json
{
  "detail": "Document '550e8400' not found."
}
```

---

## Rate Limits

No rate limits — this is a local API.
Performance depends on your hardware and Ollama model.

---

## Example — cURL

```bash
# Upload
curl -X POST http://localhost:8000/api/documents/upload \
  -F "file=@apple_ar_2024.pdf" \
  -F "company=Apple Inc." \
  -F "year=2024" \
  -F "doc_type=annual_report"

# Ask question
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What was total revenue?",
    "doc_ids": ["your-doc-id-here"],
    "include_recommendation": true
  }'

# List documents
curl http://localhost:8000/api/documents/

# Delete document
curl -X DELETE http://localhost:8000/api/documents/your-doc-id-here
```
