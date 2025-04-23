# Production RAG Platform

<p align="center">
  <strong>Production-grade Retrieval-Augmented Generation API with hybrid search, multi-format ingestion, pgvector, Redis memory, Qwen LLMs, and FastAPI.</strong>
</p>

<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/Python-3.11%2B-3776AB" alt="Python"></a>
  <a href="#"><img src="https://img.shields.io/badge/FastAPI-API-009688" alt="FastAPI"></a>
  <a href="#"><img src="https://img.shields.io/badge/PostgreSQL-16-4169E1" alt="PostgreSQL"></a>
  <a href="#"><img src="https://img.shields.io/badge/pgvector-Vector Search-336791" alt="pgvector"></a>
  <a href="#"><img src="https://img.shields.io/badge/Redis-Memory-DC382D" alt="Redis"></a>
  <a href="#"><img src="https://img.shields.io/badge/Qwen-LLM-615CED" alt="Qwen"></a>
  <a href="#"><img src="https://img.shields.io/badge/Docker-Infrastructure-2496ED" alt="Docker"></a>
</p>

<p align="center">
  <a href="https://github.com/Talha-Techie">GitHub Profile</a> ·
  <a href="#quick-start">Quick Start</a> ·
  <a href="#architecture">Architecture</a> ·
  <a href="#security">Security</a>
</p>

---

## Overview

**Production RAG Platform** is a retrieval-augmented generation backend built around FastAPI, PostgreSQL + pgvector, local sentence-transformer embeddings, Redis-backed conversation memory, and an OpenAI-compatible Qwen endpoint.

It supports multi-format document ingestion, semantic vector search, PostgreSQL full-text keyword search, hybrid retrieval, repository/document management, bulk uploads, conversation-aware answer generation, API-key authentication, and a Streamlit interface.

### Business / Engineering Value

- Hybrid vector + PostgreSQL keyword retrieval.
- Local `BAAI/bge-large-en-v1.5` embeddings with 1024 dimensions.
- PDF, TXT, MD, DOCX, and HTML document processing.
- Redis-backed conversation memory with a 7-day TTL.
- Qwen models through an OpenAI-compatible DashScope API.
- Repository/document CRUD and bulk ingestion endpoints.
- X-API-Key authentication and Swagger integration.

## Technology Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI |
| Database | PostgreSQL 16 |
| Vector search | pgvector |
| Embeddings | BAAI/bge-large-en-v1.5 |
| Memory | Redis 7 |
| LLM | Qwen via DashScope |
| Frontend | Streamlit |
| Infrastructure | Docker Compose |

---

## Key Features

- **Document Processing**: Upload and process PDF, TXT, MD, DOCX, and HTML files
- **Vector Search**: Semantic similarity search using pgvector and local sentence-transformers embeddings (BAAI/bge-large-en-v1.5, 1024-dim)
- **Keyword Search**: Full-text search using PostgreSQL GIN indexes
- **Hybrid Search**: Combines vector and keyword search for optimal results
- **Conversation Memory**: Maintains chat context using Redis-backed storage (7-day TTL)
- **LLM Integration**: Ali Cloud DashScope Qwen models via OpenAI-compatible API
- **Bulk Upload**: Upload multiple documents in parallel
- **Web UI**: Streamlit-based interface for easy interaction

## Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Streamlit     │────▶│    FastAPI      │────▶│   PostgreSQL    │
│     Frontend    │     │     Backend     │     │   + pgvector    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                │
                                ▼
                        ┌─────────────────┐     ┌─────────────────┐
                        │  Embedding      │     │      Redis      │
                        │  DashScope      │     │  Conversation   │
                        └─────────────────┘     └─────────────────┘
                                │
                                ▼
                        ┌─────────────────┐
                        │   LLM Service   │
                        │  (Qwen / Any    │
                        │  OpenAI-compat) │
                        └─────────────────┘
```

## Prerequisites

- Docker Desktop (for PostgreSQL + Redis)
- Python 3.11+
- Ali Cloud DashScope API key (for LLM only — embeddings now run locally)
- SerpAPI key (optional — web search is currently disabled)

## Quick Start

### 1. Start Infrastructure

From the project root:

```bash
docker compose up -d
```

This starts:
- PostgreSQL 16 with pgvector on port `5432`
- Redis 7 on port `6379`

Verify both are healthy:

```bash
docker compose ps
```

### 2. Configure Environment

Create `source/.env`:

```env
# Required
OPENAI_API_KEY=your_dashscope_api_key_here
SERPAPI_API_KEY=your_serpapi_key_here
API_KEY=your_secret_api_key_here

# LLM (Ali Cloud DashScope — Qwen models)
OPENAI_BASE_URL=https://dashscope-intl.aliyuncs.com/compatible-mode/v1
OPENAI_MODEL=qwen-plus

# Embeddings (local model — downloads ~1.3 GB from HuggingFace on first run)
EMBEDDING_MODEL=BAAI/bge-large-en-v1.5
EMBEDDING_DIMENSION=1024

# Database (matches docker-compose.yml defaults — no changes needed)
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=rag_db
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# Redis (matches docker-compose.yml defaults — no changes needed)
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

# Chunking
CHUNK_SIZE=1000
CHUNK_OVERLAP=200
MAX_CONVERSATION_HISTORY=10
```

> `OPENAI_API_KEY`, `SERPAPI_API_KEY`, and `API_KEY` are required — the app will not start without them.

### 3. Install Dependencies

```bash
cd source
pip install -r requirements.txt
```

### 4. Run the Application

**FastAPI backend** (from `source/`):

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

The schema (`repositories`, `documents`, `chunks`, indexes) is created automatically on first startup.

**Streamlit frontend** (separate terminal, from `source/`):

```bash
streamlit run streamlit_app.py --server.port 8501
```

### 5. Access the Application

| Service | URL |
|---------|-----|
| API Documentation | http://localhost:8000/docs |
| Streamlit UI | http://localhost:8501 |
| Health Check | http://localhost:8000/health |

## API Endpoints

### Repositories

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/repositories` | Create a new repository |
| GET | `/repositories` | List all repositories |
| GET | `/repositories/{id}` | Get repository by ID |
| PUT | `/repositories/{id}` | Update repository |
| DELETE | `/repositories/{id}` | Delete repository |

### Documents

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/documents` | Upload a single document |
| POST | `/documents/bulk` | Upload multiple documents |
| GET | `/documents` | List documents |
| GET | `/documents/{id}` | Get document by ID |
| DELETE | `/documents/{id}` | Delete document |

### Search

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/search` | Perform RAG search with LLM answer |

### Health & Debug

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Check service health |
| GET | `/debug/chunks` | View stored chunks |
| GET | `/debug/vector-search` | Test vector similarity |
| GET | `/debug/pgvector-version` | Check pgvector version |
| POST | `/debug/reprocess-embeddings` | Reprocess all embeddings |

## Authentication

All endpoints require an `X-API-Key` header. Set `API_KEY` in `source/.env` and pass it with every request.

| Response | Meaning |
|----------|---------|
| `401 Unauthorized` | Header missing |
| `403 Forbidden` | Key is wrong |

The Swagger UI at `http://localhost:8000/docs` has an **Authorize** button (lock icon) where you can enter the key once for all requests in that session.

## Usage Examples

### Upload Documents via API

```bash
# Single document
curl -X POST "http://localhost:8000/documents" \
  -H "X-API-Key: your_secret_api_key_here" \
  -F "file=@document.pdf"

# Bulk upload
curl -X POST "http://localhost:8000/documents/bulk" \
  -H "X-API-Key: your_secret_api_key_here" \
  -F "files=@doc1.pdf" \
  -F "files=@doc2.pdf"
```

### Search Query

```bash
curl -X POST "http://localhost:8000/search" \
  -H "X-API-Key: your_secret_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is the main topic?",
    "source": "vector",
    "top_k": 5,
    "use_conversation_memory": true
  }'
```

### Python Client

```python
import requests

HEADERS = {"X-API-Key": "your_secret_api_key_here"}

# Upload document
with open("document.pdf", "rb") as f:
    response = requests.post(
        "http://localhost:8000/documents",
        headers=HEADERS,
        files={"file": f}
    )
    print(response.json())

# Search
response = requests.post(
    "http://localhost:8000/search",
    headers=HEADERS,
    json={
        "query": "Summarize the document",
        "source": "vector",
        "top_k": 5
    }
)
print(response.json()["answer"])
```

## Configuration

### Chunking

- **Chunk Size**: 1000 characters (configurable via `CHUNK_SIZE`)
- **Chunk Overlap**: 200 characters (configurable via `CHUNK_OVERLAP`)

### Embedding Model

Default: `BAAI/bge-large-en-v1.5` via local sentence-transformers (1024 dimensions, ~1.3 GB, downloaded from HuggingFace on first run and cached at `~/.cache/huggingface`).

To use a different model, update `EMBEDDING_MODEL` and `EMBEDDING_DIMENSION` in `.env`. If you change dimensions, drop and recreate the `chunks` table (or run `POST /debug/reprocess-embeddings`).

### LLM Provider

Any OpenAI-compatible endpoint. Configure via `OPENAI_BASE_URL`, `OPENAI_MODEL`, and `OPENAI_API_KEY` in `.env`.

## Project Structure

```
RAG-FastAPI/
├── docker-compose.yml          # PostgreSQL 16 + pgvector, Redis 7
├── README.md
└── source/
    ├── app/
    │   ├── main.py             # FastAPI application + endpoints
    │   ├── config.py           # Pydantic settings (reads .env)
    │   ├── database.py         # Connection pool + schema init
    │   ├── models.py           # Pydantic request/response schemas
    │   ├── rag_service.py      # Hybrid search + LLM orchestration
    │   ├── embedding_service.py   # DashScope embeddings (batch=10)
    │   ├── llm_service.py         # LLM completion + conversation history
    │   ├── document_service.py    # Document upload + chunking pipeline
    │   ├── document_processor.py  # PDF/TXT/MD/DOCX/HTML extraction
    │   ├── repository_service.py  # Repository CRUD
    │   ├── conversation_memory.py # Redis conversation storage
    │   └── web_search_service.py  # SerpAPI (disabled)
    ├── migrations/
    │   └── 002_add_rbac.sql    # Optional RBAC schema (not applied)
    ├── streamlit_app.py        # Web UI
    ├── verify_setup.py         # Infrastructure connectivity check
    ├── requirements.txt
    └── uploads/                # Uploaded document storage
```

## Troubleshooting

**Docker containers conflict on startup**
```bash
docker compose down -v
docker compose up -d
```

**Database connection error**
- Verify containers are running and healthy: `docker compose ps`
- Check `POSTGRES_*` values in `source/.env`

**pgAdmin connection refused / password error**
- Run `docker compose down -v && docker compose up -d` to reset the volume with correct credentials
- Connect with: host `localhost`, port `5432`, user `postgres`, password `password`

**Embedding dimension mismatch**
- Verify `EMBEDDING_DIMENSION` in `.env` matches your model (default: `1024` for text-embedding-v4)
- Reprocess: `POST /debug/reprocess-embeddings`

**LLM API errors**
- Verify `OPENAI_API_KEY` is a valid DashScope key
- Verify `OPENAI_BASE_URL` is reachable from your machine

**App won't start — validation error**
- `OPENAI_API_KEY` and `SERPAPI_API_KEY` have no defaults and must be present in `source/.env`

---

## Security

For production use, treat uploaded documents, prompts, model outputs, credentials, user data, and tool/API responses as potentially sensitive.

Recommended controls include:

- Keep secrets in environment variables or a dedicated secret manager.
- Never commit `.env` files, API keys, database passwords, or tokens.
- Validate and constrain all external inputs before processing.
- Apply authentication and authorization to production endpoints where appropriate.
- Use least-privilege access for databases, tools, cloud resources, and service accounts.
- Enforce HTTPS/TLS at the deployment boundary.
- Add request limits, timeouts, structured logging, and dependency scanning.
- Review model/tool outputs before allowing irreversible actions.

> Security, compliance, SSO, RBAC, or enterprise governance capabilities should only be advertised when they are implemented and verified in the deployed environment.

## Production Considerations

Before operating this project in a production environment, consider adding or validating:

- Centralized logs and metrics
- Health and readiness checks
- Request tracing and correlation IDs
- Rate limiting and abuse controls
- Persistent state and backup strategy
- CI/CD quality gates
- Dependency and container vulnerability scanning
- Model/LLM latency, reliability, and cost monitoring where applicable
- Horizontal scaling and externalized state where required

## Contributing

Contributions are welcome.

```bash
git checkout -b feature/your-feature
git add .
git commit -m "feat: describe your change"
git push origin feature/your-feature
```

When opening a pull request, include the motivation, implementation summary, testing performed, and any API or architecture implications.

## Maintainer

Maintained by **Talha-Techie**.

- GitHub: [github.com/Talha-Techie](https://github.com/Talha-Techie)

## License

Refer to the repository's `LICENSE` file or the license section above for the applicable terms.

---

<p align="center">
  <strong>Designed as a clean, modular, production-oriented AI/ML engineering project.</strong>
</p>
