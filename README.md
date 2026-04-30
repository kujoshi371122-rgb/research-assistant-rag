# Research Assistant — Local RAG Fullstack
## TEST LINE

A local, privacy-first RAG (Retrieval-Augmented Generation) research assistant.
Upload documents, generate embeddings, and ask questions grounded in your own sources — fully offline using Ollama.

---

## Architecture

```
research-assistant-fullstack/
├── backend/                     FastAPI + SQLAlchemy async
│   ├── api/
│   │   ├── conversations.py     Chat + conversation CRUD
│   │   ├── documents.py         Document upload, CRUD, embed trigger
│   │   ├── search.py            Semantic search endpoint
│   │   └── system.py            Health + status endpoint
│   ├── core/
│   │   ├── config.py            Pydantic settings from .env
│   │   └── database.py          Async SQLAlchemy engine + session
│   ├── models/
│   │   └── models.py            Conversation, Message, Document, Chunk
│   ├── schemas/
│   │   └── schemas.py           Pydantic request/response models
│   ├── services/
│   │   ├── ollama_service.py    Ollama HTTP client (generate + embed)
│   │   └── rag_service.py       Chunking, cosine similarity, retrieval
│   ├── main.py                  FastAPI app + CORS + lifespan
│   └── requirements.txt
├── frontend/                    React + Vite
│   ├── src/
│   │   ├── components/
│   │   │   ├── chat/            ChatView (messages, sources, RAG toggle)
│   │   │   ├── documents/       DocumentsView (upload, embed, list)
│   │   │   ├── search/          SearchView + SystemView
│   │   │   ├── layout/          Sidebar navigation
│   │   │   └── ui/              Toast, shared UI
│   │   ├── hooks/               useData hooks (conversations, docs, status)
│   │   ├── services/            axios API client (api.js)
│   │   ├── stores/              Zustand global store
│   │   ├── utils/               helpers (timeAgo, formatBytes, etc.)
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css            Dark dashboard theme
│   ├── index.html
│   ├── package.json
│   └── vite.config.js           Proxy /api → localhost:8000
├── uploads/                     Uploaded files (git-ignored)
├── .env.example
└── README.md
```

---

## Prerequisites

| Tool | Purpose |
|------|---------|
| Python 3.10+ | Backend runtime |
| Node.js 18+ | Frontend build |
| Ollama | Local LLM inference |

---

## Setup & Run

### 1. Clone / unzip

```bash
cd research-assistant-fullstack
```

### 2. Configure environment

```bash
cp .env.example .env
# Edit .env if you want different models or DB
```

### 3. Start Ollama and pull models

```bash
# Install Ollama from https://ollama.ai
ollama serve                         # keep running in a terminal

ollama pull mistral                  # chat model (~4GB)
ollama pull nomic-embed-text         # embedding model (~275MB)
```

Lighter alternatives if disk is limited:
```bash
ollama pull orca-mini                # ~2GB chat model
ollama pull all-minilm               # ~45MB embedding model
# Then update .env: OLLAMA_MODEL=orca-mini  OLLAMA_EMBEDDING_MODEL=all-minilm
```

### 4. Backend

```bash
cd research-assistant-fullstack

# Create venv
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r backend/requirements.txt

# Run server (from project root)
uvicorn backend.main:app --reload --host 0.0.0.0 --port 8000
```

Backend will be at: http://localhost:8000
API docs: http://localhost:8000/docs

### 5. Frontend

```bash
cd frontend
npm install
npm run dev
```

Frontend will be at: http://localhost:5173

---

## End-to-End Test Flow

1. Open http://localhost:5173
2. Click **System** in the sidebar → confirm all services show green
3. Click **Documents** → **Add text** → paste some content → Save
4. Click **⚡ Embed** on the document → wait for "Embedded N chunks"
5. Click **Search** → type a query related to your document → verify results
6. Click **Chat** → **+ New Chat** → ask a question about your document
7. Observe grounded answer with source chunks and confidence badge

---

## API Reference

| Method | Path | Description |
|--------|------|-------------|
| GET | /api/conversations | List all conversations |
| POST | /api/conversations | Create conversation |
| PATCH | /api/conversations/{id} | Rename |
| DELETE | /api/conversations/{id} | Delete |
| GET | /api/conversations/{id}/messages | Get messages |
| POST | /api/chat | Send message (RAG-enabled) |
| GET | /api/documents | List documents |
| POST | /api/documents | Create from text |
| POST | /api/documents/upload | Upload file |
| DELETE | /api/documents/{id} | Delete |
| POST | /api/documents/{id}/embed | Generate embeddings |
| POST | /api/search | Semantic search |
| GET | /api/system/status | Health + stats |

---

## Configuration

All settings via `.env`:

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `sqlite+aiosqlite:///./research_assistant.db` | DB connection |
| `OLLAMA_BASE_URL` | `http://localhost:11434` | Ollama API |
| `OLLAMA_MODEL` | `mistral` | Chat/generation model |
| `OLLAMA_EMBEDDING_MODEL` | `nomic-embed-text` | Embedding model |
| `OLLAMA_TIMEOUT` | `120` | Request timeout (seconds) |
| `MAX_CHUNK_SIZE` | `500` | Words per chunk |
| `CHUNK_OVERLAP` | `50` | Overlap between chunks |
| `TOP_K_RESULTS` | `5` | Default retrieval count |
| `LOCAL_MODE` | `true` | Skip auth (local single-user) |

---

## Troubleshooting

**Blank screen / API errors** — Make sure backend is running on port 8000. Check browser console.

**Ollama unavailable** — Run `ollama serve` and confirm with `curl http://localhost:11434/api/tags`

**Slow first response** — First LLM call loads the model into VRAM. Subsequent calls are faster.

**Embedding error** — Pull the embedding model: `ollama pull nomic-embed-text`

**CORS error** — The Vite dev server proxies `/api` to port 8000 automatically.
