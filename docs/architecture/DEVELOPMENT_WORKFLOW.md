# Development Workflow — Ticker

> Last updated: 2026-10-10 (Phase 0 — Foundation)

---

## 1. Environment setup

### Requirements

- Python 3.10 or later
- Git (with user identity configured)
- A Groq API key (free tier: https://console.groq.com/)

### First-time setup

```powershell
# Clone (already done — skip if Ticker/ exists)
git clone https://github.com/Max0786234/Ticker.git
cd Ticker

# Create virtual environment
python -m venv .venv

# Activate (Windows PowerShell)
.venv\Scripts\Activate.ps1

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
copy .env.example .env
# Open .env and add your GROQ_API_KEY
```

### Verifying setup

```powershell
python --version              # Should be 3.10+
pip show fastapi              # Should show fastapi 0.115.x
pip show groq                 # Should show groq 0.12.x
pip show scikit-learn         # Should show scikit-learn 1.5.x
```

---

## 2. Running the application

### Terminal 1 — FastAPI backend

```powershell
# From the Ticker/ directory with .venv active
uvicorn app.main:app --reload --port 8000
```

- API base: http://localhost:8000
- Swagger docs: http://localhost:8000/docs
- Redoc: http://localhost:8000/redoc

### Terminal 2 — Streamlit frontend

```powershell
streamlit run frontend/streamlit_app.py
```

- UI: http://localhost:8501

### Seeding the database (run once)

```powershell
python -m app.db.seed
```

Safe to run multiple times — idempotent.

---

## 3. Running tests

```powershell
# All tests
pytest tests/ -v

# Specific test file
pytest tests/test_retrieval.py -v

# With output (no capture)
pytest tests/ -v -s

# Run only fast unit tests (exclude slow integration tests)
pytest tests/ -v -m "not integration"
```

Tests use a separate SQLite database (`test_ticker.db` or in-memory) and mock the Groq API. A real API key is not required for the test suite.

---

## 4. Git workflow

### Standard feature branch flow

```powershell
# Start from main
git checkout main
git pull origin main

# Create a feature branch
git checkout -b feature/database-seed

# ... make changes ...

# Stage specific files (preferred over git add .)
git add app/db/models.py app/db/database.py app/db/seed.py

# Commit with descriptive message
git commit -m "feat: add HistoricalTicket model and idempotent seeder"

# Push branch
git push origin feature/database-seed

# After testing, merge to main
git checkout main
git merge feature/database-seed
git push origin main

# Delete the feature branch
git branch -d feature/database-seed
git push origin --delete feature/database-seed
```

### Before any commit

```powershell
# Check what is staged
git status
git diff --staged

# Confirm no secrets are staged
git diff --staged | Select-String "GROQ_API_KEY|sk-"
```

### If you accidentally stage .env

```powershell
git reset HEAD .env
```

---

## 5. Branch naming reference

| Prefix     | Use for                              | Example                        |
|------------|--------------------------------------|--------------------------------|
| `chore/`   | Infrastructure, tooling, docs        | `chore/project-foundation`     |
| `feature/` | New functionality                    | `feature/database-seed`        |
| `fix/`     | Bug fixes                            | `fix/retrieval-empty-corpus`   |
| `test/`    | Cross-component test suites          | `test/end-to-end-acceptance`   |

---

## 6. Phase-by-phase development plan

| Phase | Branch                       | Deliverable                                    |
|-------|------------------------------|------------------------------------------------|
| 0     | `chore/project-foundation`   | Foundation files, docs, Git history            |
| 1     | `feature/database-seed`      | DB models, 50 synthetic tickets, seeder        |
| 2     | `feature/nlp-retrieval`      | TF-IDF service, cosine similarity, tests       |
| 3     | `feature/groq-generation`    | Groq client, prompt, 5-step validation, tests  |
| 4     | `feature/fastapi-api`        | Routes, schemas, orchestration, tests          |
| 5     | `feature/streamlit-ui`       | Frontend, api_client, real API integration     |
| 6     | `test/end-to-end-acceptance` | Full pipeline tests, error path coverage       |
| 7     | `chore/demo-prep`            | Demo script, Q&A prep, optional improvements   |

---

## 7. Mandatory update protocol (for AI agents)

### At the start of each significant task

1. Read `PROJECT_CONTEXT.md`.
2. Read `RECOVERY.md` and `TASK_TRACKER.md`.
3. Read the latest entries in `IMPLEMENTATION_LOG.md`.
4. Run `git status` and inspect relevant code files.

### At the end of each significant task

1. Run the relevant tests.
2. Update `TASK_TRACKER.md` with current status.
3. Append a new entry to `IMPLEMENTATION_LOG.md`.
4. Update `PROJECT_CONTEXT.md` if architecture, interfaces, or status changed.
5. Update `RECOVERY.md` with the latest verified state and next safe action.
6. Verify that all four continuity files agree with each other.

### Before any risky operation

Record the current state. Create a Git checkpoint commit. Preserve working code.

---

## 8. Debugging tips

### FastAPI not starting

```powershell
# Check if port 8000 is in use
netstat -ano | findstr :8000

# Run without reload to see startup errors
uvicorn app.main:app --port 8000
```

### SQLite database issues

```powershell
# Delete and reseed (safe — re-runnable)
Remove-Item ticker.db -ErrorAction SilentlyContinue
python -m app.db.seed
```

### Groq API errors

- Check that `GROQ_API_KEY` is set in `.env`.
- Verify the model name in `GROQ_MODEL` against https://console.groq.com/docs/models.
- Check the Groq status page: https://status.groq.com/

### Test failures

```powershell
# Run with full output to see print statements
pytest tests/ -v -s

# Run a single test function
pytest tests/test_retrieval.py::test_cosine_similarity_ranking -v
```
