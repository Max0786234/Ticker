# Ticker 🎫

**AI-Powered IT Ticket Resolution Suggestion Engine**

Ticker is a full-stack IT support assistant that uses historical ticket data and a hybrid NLP pipeline to suggest troubleshooting steps for new IT problems.

When a user submits an issue (e.g. *"My VPN keeps disconnecting after the Windows update"*), Ticker:

1. Searches ~50 historical support tickets using **TF-IDF + Cosine Similarity** (Scikit-learn)
2. Retrieves the most relevant tickets and their recorded resolutions
3. Passes that evidence to the **Groq LLaMA API** as context
4. Returns **exactly five actionable resolution steps** synthesized from the historical evidence
5. Shows which historical tickets supported the answer

This is a **Retrieval-Augmented Generation (RAG)** workflow built with lexical retrieval — no embeddings or vector databases required for the MVP.

---

## Architecture overview

```
Streamlit UI  →  FastAPI  →  Orchestration Service
                                ├── SQLAlchemy / SQLite  (historical tickets)
                                ├── TF-IDF Retrieval  (Scikit-learn)
                                └── Groq LLM  (LLaMA 3.x)
```

See [`docs/architecture/ARCHITECTURE.md`](docs/architecture/ARCHITECTURE.md) for the full diagram and component descriptions.

---

## Quick start

### Prerequisites

- Python 3.10 or later
- A [Groq API key](https://console.groq.com/) (free tier is sufficient for development)
- Git

### 1. Clone and enter the project

```bash
git clone https://github.com/Max0786234/Ticker.git
cd Ticker
```

### 2. Create a virtual environment

```bash
python -m venv .venv

# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

```bash
cp .env.example .env
# Edit .env and add your GROQ_API_KEY
```

### 5. Initialise and seed the database

```bash
python -m app.db.seed
```

This creates `ticker.db` with approximately 50 synthetic historical tickets. Running it again is safe — the seed is idempotent.

### 6. Start the FastAPI backend

```bash
uvicorn app.main:app --reload --port 8000
```

The API documentation is available at: **http://localhost:8000/docs**

### 7. Start the Streamlit frontend (new terminal)

```bash
streamlit run frontend/streamlit_app.py
```

The UI opens at: **http://localhost:8501**

---

## Running tests

```bash
pytest tests/ -v
```

Tests use a separate temporary SQLite database and mock the Groq API — no real API key is needed for the test suite.

---

## Project structure

```
Ticker/
├── app/
│   ├── main.py                 # FastAPI application entry point
│   ├── config.py               # Settings loaded from .env
│   ├── api/
│   │   ├── routes.py           # HTTP endpoint definitions
│   │   └── schemas.py          # Pydantic request / response models
│   ├── db/
│   │   ├── database.py         # SQLAlchemy engine and session factory
│   │   ├── models.py           # HistoricalTicket ORM model
│   │   └── seed.py             # Idempotent database seeder
│   └── services/
│       ├── retrieval.py        # TF-IDF + cosine-similarity retrieval
│       ├── llm_service.py      # Groq API client and prompt logic
│       └── resolution_service.py  # Orchestration layer
├── frontend/
│   ├── streamlit_app.py        # Streamlit user interface
│   └── api_client.py           # HTTP client for FastAPI
├── data/
│   └── synthetic_tickets.json  # Source data for database seeding
├── tests/
├── docs/
│   ├── architecture/
│   │   ├── ARCHITECTURE.md
│   │   ├── API_CONTRACT.md
│   │   ├── DATABASE_SCHEMA.md
│   │   └── DEVELOPMENT_WORKFLOW.md
│   └── learning/
│       └── README.md
├── PROJECT_CONTEXT.md
├── IMPLEMENTATION_LOG.md
├── RECOVERY.md
├── TASK_TRACKER.md
├── CONTRIBUTING.md
├── requirements.txt
├── .env.example
└── .gitignore
```

---

## API endpoints

| Method | Path       | Description                                      |
|--------|------------|--------------------------------------------------|
| GET    | /health    | Returns application operational status           |
| GET    | /tickets   | Lists historical tickets (paginated)             |
| POST   | /tickets   | Creates and persists a new historical ticket     |
| POST   | /resolve   | Accepts an issue and returns five resolution steps |
| GET    | /docs      | Interactive FastAPI / Swagger documentation      |

See [`docs/architecture/API_CONTRACT.md`](docs/architecture/API_CONTRACT.md) for full schemas.

---

## Technology stack

| Layer       | Technology                                         |
|-------------|----------------------------------------------------|
| Frontend    | Streamlit                                          |
| Backend API | FastAPI + Uvicorn                                  |
| ORM         | SQLAlchemy 2.0                                     |
| Database    | SQLite (local dev) / PostgreSQL/NeonDB (optional)  |
| Retrieval   | Scikit-learn TF-IDF + cosine similarity            |
| Generation  | Groq Python SDK — LLaMA 3.x                        |
| Testing     | Pytest + HTTPX                                     |

---

## Current status

> **Phase 0 — Foundation complete.**  
> Implementation begins at Phase 1 (database models and seed data).

See [TASK_TRACKER.md](TASK_TRACKER.md) for the live status of each deliverable.

---

## Contributing

This is a single-developer hackathon project. See [CONTRIBUTING.md](CONTRIBUTING.md) for branch conventions and development workflow.

---

## License

MIT
