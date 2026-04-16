# 🤖 IoT & AI Technical Mentor Chatbot

> A RAG-powered AI assistant that answers deep technical questions about IoT, microcontrollers, sensors, Linux, and Python/ML — built with LangChain, ChromaDB, FastAPI, and a vanilla JS chat widget.

**Project P006 · RAIOT Project Arena 2026**  
Team: Ujjwal, Lakshya · Mentors: Tarun & Tapas

---

## What it does

Ask it anything like:

- *"How does I2C differ from SPI?"*
- *"Show me ESP32 code to read a DHT11 sensor"*
- *"What Linux command lists all running processes sorted by CPU?"*
- *"Explain attention mechanisms in transformers"*

It searches your own uploaded datasheets and documentation (stored in ChromaDB), feeds the relevant chunks to an LLM, and returns a precise, sourced answer — not a generic web search result.

---

## Tech Stack

| Layer | Technology |
|---|---|
| AI / RAG | LangChain + OpenAI GPT-4o-mini |
| Vector DB | ChromaDB |
| Backend API | FastAPI + Uvicorn |
| Frontend | Vanilla HTML / CSS / JS (embeddable widget) |
| Config | Pydantic Settings + `.env` |

---

## Project Structure

```
p006-iot-mentor/
├── app/
│   ├── main.py              # FastAPI app, CORS, router registration
│   ├── config.py            # Pydantic settings from .env
│   ├── routes/
│   │   └── chat.py          # POST /api/chat endpoint
│   └── services/
│       └── rag_chain.py     # LangChain RAG pipeline + ChromaDB retriever
├── docs/                    # Raw PDFs and markdown datasheets to ingest
├── scripts/
│   └── ingest.py            # One-time script to embed docs into ChromaDB
├── frontend/
│   └── widget.html          # Self-contained embeddable chat widget
├── .env.example             # Environment variable template
├── requirements.txt
└── README.md
```

---

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/ujjwal-tiwari3039/p006-iot-mentor.git
cd p006-iot-mentor
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Set up environment variables

```bash
cp .env.example .env
# Open .env and add your OpenAI API key
```

Your `.env` file:

```env
OPENAI_API_KEY=sk-...
CHROMA_PERSIST_DIR=app/db/chroma_store
COLLECTION_NAME=iot_docs
MODEL_NAME=gpt-4o-mini
TOP_K_RESULTS=5
ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:5500
```

### 4. Ingest your documentation

Drop your PDFs and markdown files into the `docs/` folder, then run:

```bash
python scripts/ingest.py
```

This chunks the documents, generates embeddings, and saves them to ChromaDB.

### 5. Start the backend

```bash
uvicorn app.main:app --reload --port 8000
```

Check it's running:

```bash
curl http://localhost:8000/health
# {"status": "ok", "model": "gpt-4o-mini"}
```

### 6. Open the chat widget

Open `frontend/widget.html` directly in your browser, or embed it in any webpage (see Embedding below).

---

## API Reference

### `POST /api/chat`

Send a question and get an AI-generated answer sourced from your documentation.

**Request body:**

```json
{
  "question": "What is the I2C protocol?",
  "history": [
    { "role": "user", "content": "previous question" },
    { "role": "assistant", "content": "previous answer" }
  ]
}
```

**Response:**

```json
{
  "answer": "I2C (Inter-Integrated Circuit) is a two-wire serial protocol...",
  "sources": ["esp32-datasheet.pdf", "linux-commands.md"]
}
```

**Other endpoints:**

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Server health check |
| GET | `/docs` | Swagger UI (interactive API docs) |
| GET | `/redoc` | ReDoc API documentation |

---

## Embedding the Widget

The chat widget is a single self-contained HTML snippet. Copy the marked section from `frontend/widget.html` into any webpage:

```html
<!-- paste into any HTML page -->
<style> /* widget styles */ </style>
<div id="p006-widget"> ... </div>
<script> ... </script>
```

To point it at your backend, update this one line in the script:

```js
const API_URL = 'http://localhost:8000/api/chat';
// Change to your deployed URL when going live
```

---

## How the RAG Pipeline Works

```
User question
     │
     ▼
FastAPI /api/chat
     │
     ▼
LangChain retriever
  → searches ChromaDB for top-K similar document chunks
     │
     ▼
Chunks + question → LLM prompt
     │
     ▼
GPT-4o-mini generates answer
     │
     ▼
Response: { answer, sources }
```

The LLM is instructed to answer only from the retrieved documentation and to say so if the docs don't cover the question — preventing hallucination.

---

## Requirements

```
fastapi==0.111.0
uvicorn[standard]==0.29.0
pydantic-settings==2.2.1
langchain==0.2.5
langchain-openai==0.1.8
langchain-chroma==0.1.1
chromadb==0.5.3
python-dotenv==1.0.1
```

Python 3.10+ recommended.

---

## Team

| Name | Role |
|---|---|
| Ujjwal | FastAPI backend, API routes, CORS config |
| Lakshya | LangChain RAG pipeline, ChromaDB, document ingestion |
| Tarun | Senior mentor |
| Tapas | Senior mentor |

---

## License

MIT License — free to use, modify, and distribute.
