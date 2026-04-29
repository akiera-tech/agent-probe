# 🔍 Agent Probe

<div align="center">

![GitHub license](https://img.shields.io/github/license/akiera-tech/agent-probe)
![GitHub stars](https://img.shields.io/github/stars/akiera-tech/agent-probe)
![GitHub issues](https://img.shields.io/github/issues/akiera-tech/agent-probe)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688)

**A lightweight, open-source tool to monitor, debug, trace, and test AI agents powered by Anthropic Claude.**

[Features](#-features) • [Installation](#-installation) • [Usage](#-usage) • [API Docs](#-api-reference) • [Contributing](#-contributing) • [License](#-license)

</div>

---

## 📖 Overview

**Agent Probe** is an open-source observability and debugging toolkit for AI agent applications built with [Anthropic Claude](https://www.anthropic.com). It helps developers inspect LLM API calls, trace agent workflows, log responses, and read/process document data — all through a clean FastAPI backend with PostgreSQL persistence.

If you're building autonomous agents, RAG pipelines, or Claude-powered apps, Agent Probe gives you full visibility into what your agent is doing and why.

---

## ✨ Features

- 🤖 **Claude Agent Monitoring** — inspect every API call made to Anthropic Claude in real time
- 🧪 **LLM Call Testing** — test prompts, compare responses, and validate agent outputs
- 🔁 **Agent Workflow Tracing** — log and trace multi-step agent runs end to end
- 📄 **Document Data Reading** — upload and extract structured data from documents for agent consumption
- 🗄️ **PostgreSQL Persistence** — store all agent sessions, traces, and logs durably
- ⚡ **FastAPI Backend** — async, high-performance REST API with auto-generated Swagger docs
- 🔍 **Probe Sessions** — group related agent calls into sessions for easier debugging
- 📊 **Response Analytics** — track token usage, latency, and model performance over time
- 🛡️ **Input/Output Validation** — validate agent inputs and outputs with Pydantic schemas
- 🧩 **Modular & Extensible** — easy to extend with new agent types or LLM providers

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.10+ |
| Framework | FastAPI |
| LLM Provider | Anthropic Claude (claude-3, claude-sonnet, etc.) |
| Database | PostgreSQL |
| ORM | SQLAlchemy + Alembic |
| Validation | Pydantic v2 |
| Server | Uvicorn |
| Document Parsing | PyMuPDF / pdfplumber |

---

## 🚀 Installation

### Prerequisites

- Python >= 3.10
- PostgreSQL (local or hosted e.g. Supabase, Neon, Railway)
- Anthropic API key — get one at [console.anthropic.com](https://console.anthropic.com)

### Clone & Install

```bash
git clone https://github.com/akiera-tech/agent-probe.git
cd agent-probe

# Create virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Environment Setup

```bash
cp .env.example .env
```

```env
# Anthropic
ANTHROPIC_API_KEY=sk-ant-your-key-here
CLAUDE_MODEL=claude-sonnet-4-5        # or claude-opus-4, claude-haiku-4-5

# PostgreSQL
DATABASE_URL=postgresql://user:password@localhost:5432/agent_probe

# Server
HOST=0.0.0.0
PORT=8000
DEBUG=true
```

### Database Setup

```bash
# Run migrations
alembic upgrade head
```

### Start the Server

```bash
# Development (with auto-reload)
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Production
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4
```

Once running, visit:
- **Swagger UI** → http://localhost:8000/docs
- **ReDoc** → http://localhost:8000/redoc

---

## 📡 API Reference

### `POST /api/probe/run`

Run a Claude agent call and log the full trace.

```bash
curl -X POST http://localhost:8000/api/probe/run \
  -H "Content-Type: application/json" \
  -d '{
    "session_id": "session-001",
    "prompt": "Summarize the uploaded document",
    "model": "claude-sonnet-4-5",
    "max_tokens": 1024
  }'
```

**Response:**
```json
{
  "success": true,
  "data": {
    "trace_id": "trace-uuid",
    "session_id": "session-001",
    "response": "The document covers...",
    "model": "claude-sonnet-4-5",
    "input_tokens": 312,
    "output_tokens": 128,
    "latency_ms": 843
  }
}
```

---

### `POST /api/probe/document`

Upload a document for Agent Probe to read and extract structured data.

```bash
curl -X POST http://localhost:8000/api/probe/document \
  -F "file=@report.pdf" \
  -F "session_id=session-001"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "document_id": "doc-uuid",
    "filename": "report.pdf",
    "pages": 12,
    "extracted_text": "...",
    "session_id": "session-001"
  }
}
```

---

### `GET /api/probe/sessions`

List all agent probe sessions with pagination.

```bash
curl "http://localhost:8000/api/probe/sessions?limit=20&page=1"
```

| Query Param | Type | Default | Description |
|---|---|---|---|
| `limit` | int | 20 | Results per page |
| `page` | int | 1 | Page number |

---

### `GET /api/probe/sessions/{session_id}/traces`

Get all traces for a specific agent session.

```bash
curl http://localhost:8000/api/probe/sessions/session-001/traces
```

---

### `GET /api/health`

Health check for server, database, and Anthropic API connectivity.

```bash
curl http://localhost:8000/api/health
```

**Response:**
```json
{
  "status": "ok",
  "db": "connected",
  "anthropic": "reachable",
  "version": "0.1.0"
}
```

---

## 📁 Project Structure

```
agent-probe/
├── app/
│   ├── api/
│   │   └── routes/
│   │       ├── probe.py           # Agent probe endpoints
│   │       └── health.py          # Health check
│   ├── controllers/
│   │   └── probe_controller.py    # Business logic
│   ├── services/
│   │   ├── claude_service.py      # Anthropic Claude integration
│   │   └── document_service.py    # Document parsing & extraction
│   ├── models/
│   │   ├── session.py             # SQLAlchemy session model
│   │   └── trace.py               # SQLAlchemy trace model
│   ├── schemas/
│   │   └── probe.py               # Pydantic request/response schemas
│   └── db/
│       └── database.py            # PostgreSQL connection & session
├── alembic/                       # DB migrations
├── tests/
├── .env.example
├── requirements.txt
├── main.py
└── README.md
```

---

## 🤝 Contributing

Agent Probe is community-driven and open to all contributors — whether it's a bug fix, new feature, or documentation improvement.

### How to Contribute

1. **Fork** the repository on GitHub
2. **Clone** your fork locally
   ```bash
   git clone https://github.com/YOUR_USERNAME/agent-probe.git
   cd agent-probe
   ```
3. **Create** a feature branch
   ```bash
   git checkout -b feat/your-feature-name
   ```
4. **Set up** your dev environment
   ```bash
   python -m venv venv && source venv/bin/activate
   pip install -r requirements.txt
   cp .env.example .env   # add your test credentials
   ```
5. **Commit** using [Conventional Commits](https://www.conventionalcommits.org/)
   ```bash
   git commit -m "feat: add token usage analytics"
   git commit -m "fix: handle empty Claude response"
   git commit -m "docs: add document upload example"
   ```
6. **Push** and open a Pull Request against `main`

### What We're Looking For

- 🐛 Bug fixes with reproduction steps
- ✨ New agent tracing or observability features
- 🤖 Additional LLM provider support (OpenAI, Gemini, Mistral)
- 📄 More document format support (DOCX, CSV, HTML)
- 🧪 Unit and integration tests
- 📝 Documentation improvements and example notebooks

### Code Standards

- Follow [PEP 8](https://peps.python.org/pep-0008/) style
- Use type hints on all functions
- All endpoints must use `async/await`
- Add Pydantic schemas for all request/response bodies
- Write tests for new features in `tests/`

### Reporting Bugs

Open a GitHub Issue with:
- Python version (`python --version`)
- Steps to reproduce
- Expected vs actual behavior
- Full error traceback

---

## 📄 License

MIT License — see [LICENSE](./LICENSE) for full details.

Free to use, modify, and distribute. Attribution appreciated but not required.

---

## 🌟 Support the Project

If Agent Probe helps your AI development workflow, please ⭐ star the repo — it helps others discover it!

---

<div align="center">
Made with ❤️ by <a href="https://github.com/akiera-tech/agent-probe">akiera-tech</a> and contributors
</div>
