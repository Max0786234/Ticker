# TASK_TRACKER.md — Ticker

> **Allowed status values:**
> `NOT STARTED` | `IN PROGRESS` | `BLOCKED` | `IMPLEMENTED — NOT VERIFIED` | `VERIFIED` | `DEFERRED`
>
> A task cannot be marked **VERIFIED** without a successful test run or documented manual verification.
> Record the verification evidence in the Notes column.

---

## Phase 0 — Project Foundation

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| Repository initialised (GitHub) | IN PROGRESS | Cloned empty repo; initial commit pending              |
| Foundation branch created     | IN PROGRESS | `chore/project-foundation` to be created after init commit |
| `.gitignore` and `.env.example` | IMPLEMENTED — NOT VERIFIED | Files created; push pending |
| `requirements.txt`            | IMPLEMENTED — NOT VERIFIED | Versions pinned; install not yet verified in clean env |
| `README.md`                   | IMPLEMENTED — NOT VERIFIED | Content created; not yet rendered on GitHub            |
| `CONTRIBUTING.md`             | IMPLEMENTED — NOT VERIFIED | Content created                                        |
| `PROJECT_CONTEXT.md`          | IN PROGRESS | Being written in this session                          |
| `IMPLEMENTATION_LOG.md`       | IN PROGRESS | Being written in this session                          |
| `RECOVERY.md`                 | IN PROGRESS | Being written in this session                          |
| `TASK_TRACKER.md`             | IN PROGRESS | This file                                              |
| Architecture document         | IMPLEMENTED — NOT VERIFIED | `docs/architecture/ARCHITECTURE.md` created           |
| API contract document         | IMPLEMENTED — NOT VERIFIED | `docs/architecture/API_CONTRACT.md` created            |
| Database schema document      | IMPLEMENTED — NOT VERIFIED | `docs/architecture/DATABASE_SCHEMA.md` created         |
| Development workflow document | IMPLEMENTED — NOT VERIFIED | `docs/architecture/DEVELOPMENT_WORKFLOW.md` created    |
| Learning documentation        | IMPLEMENTED — NOT VERIFIED | `docs/learning/README.md` created                      |
| Directory structure created   | IMPLEMENTED — NOT VERIFIED | All `app/`, `frontend/`, `data/`, `tests/` dirs        |
| `__init__.py` stubs           | IMPLEMENTED — NOT VERIFIED | Python package markers created                         |
| Foundation committed and pushed | NOT STARTED | Pending file creation completion                       |

---

## Phase 1 — Database Models and Seed Data

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| `app/config.py` (Pydantic Settings) | NOT STARTED |                                                  |
| `app/db/database.py`          | NOT STARTED |                                                        |
| `app/db/models.py` (HistoricalTicket) | NOT STARTED |                                                 |
| `data/synthetic_tickets.json` (~50 tickets) | NOT STARTED |                                         |
| `app/db/seed.py` (idempotent) | NOT STARTED |                                                        |
| `tests/test_seed.py`          | NOT STARTED |                                                        |
| Phase 1 tests pass            | NOT STARTED |                                                        |

---

## Phase 2 — TF-IDF Retrieval

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| `app/services/retrieval.py`   | NOT STARTED |                                                        |
| TF-IDF vectoriser fit/transform | NOT STARTED |                                                      |
| Cosine similarity ranking     | NOT STARTED |                                                        |
| Edge case handling (empty corpus, OOV) | NOT STARTED |                                              |
| `tests/test_retrieval.py`     | NOT STARTED |                                                        |
| Phase 2 tests pass            | NOT STARTED |                                                        |

---

## Phase 3 — Groq LLM Integration

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| `app/services/llm_service.py` | NOT STARTED |                                                        |
| Prompt construction           | NOT STARTED |                                                        |
| Exactly-5-step validation     | NOT STARTED |                                                        |
| Retry on invalid format       | NOT STARTED |                                                        |
| Groq error handling           | NOT STARTED |                                                        |
| `tests/test_llm_service.py`   | NOT STARTED | Must mock Groq API                                     |
| Live Groq call verified       | NOT STARTED | Separate from unit tests; requires real API key        |
| Phase 3 tests pass            | NOT STARTED |                                                        |

---

## Phase 4 — Resolution Orchestration and FastAPI Endpoints

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| `app/services/resolution_service.py` | NOT STARTED |                                               |
| `app/api/schemas.py`          | NOT STARTED |                                                        |
| `app/api/routes.py`           | NOT STARTED |                                                        |
| `app/main.py`                 | NOT STARTED |                                                        |
| GET /health                   | NOT STARTED |                                                        |
| GET /tickets                  | NOT STARTED |                                                        |
| POST /tickets                 | NOT STARTED |                                                        |
| POST /resolve                 | NOT STARTED |                                                        |
| FastAPI /docs renders         | NOT STARTED |                                                        |
| `tests/test_api_routes.py`    | NOT STARTED |                                                        |
| `tests/test_resolution_service.py` | NOT STARTED |                                                  |
| Phase 4 tests pass            | NOT STARTED |                                                        |

---

## Phase 5 — Streamlit Frontend

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| `frontend/api_client.py`      | NOT STARTED |                                                        |
| `frontend/streamlit_app.py`   | NOT STARTED |                                                        |
| Issue submission form         | NOT STARTED |                                                        |
| Resolution display (5 steps)  | NOT STARTED |                                                        |
| Matched tickets display       | NOT STARTED |                                                        |
| Error / loading states        | NOT STARTED |                                                        |
| Phase 5 manual verification   | NOT STARTED |                                                        |

---

## Phase 6 — End-to-End Acceptance Tests

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| `tests/conftest.py` complete  | NOT STARTED |                                                        |
| Full pipeline integration test | NOT STARTED |                                                       |
| Insufficient evidence path tested | NOT STARTED |                                                    |
| Generation error path tested  | NOT STARTED |                                                        |
| Empty input validation tested | NOT STARTED |                                                        |
| All tests pass                | NOT STARTED |                                                        |

---

## Phase 7 — Demo Preparation and Optional Enhancements

| Task                          | Status      | Notes / Verification Evidence                          |
|-------------------------------|-------------|--------------------------------------------------------|
| Demo script prepared          | NOT STARTED |                                                        |
| Hackathon Q&A answers prepared | NOT STARTED |                                                       |
| PostgreSQL/NeonDB migration   | DEFERRED    | Optional; not required for MVP                         |
| Semantic embedding upgrade    | DEFERRED    | Optional; not required for MVP                         |
| UI polish                     | DEFERRED    | Optional                                               |

---

*Last updated: 2026-10-10 (Phase 0 — Foundation)*
