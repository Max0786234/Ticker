# Contributing to Ticker

This is a single-developer hackathon project. These conventions exist to keep the history clean and the work recoverable — not to enforce process for its own sake.

---

## Git workflow

### Branch naming

| Prefix              | Purpose                                          | Example                        |
|---------------------|--------------------------------------------------|--------------------------------|
| `chore/`            | Project infrastructure, tooling, documentation  | `chore/project-foundation`     |
| `feature/`          | New functionality                               | `feature/database-seed`        |
| `fix/`              | Bug fixes                                       | `fix/retrieval-empty-corpus`   |
| `test/`             | Tests that span multiple components             | `test/end-to-end-acceptance`   |

### Commit message format

```
<type>: <short summary in present tense>

<optional body — explain WHY, not WHAT>
```

Types: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `style`

Examples:

```
feat: add TF-IDF cosine similarity retrieval service
fix: handle empty corpus edge case in retrieval
chore: add .gitignore and environment template
docs: document API_CONTRACT with Pydantic schemas
```

### Branch lifecycle

1. Create branch from `main`.
2. Commit incremental, reviewable changes.
3. Run `pytest` before merging.
4. Merge to `main` — squash only if the branch has many noisy WIP commits.
5. Delete the branch after merge.

### Things never to do

- Do not force-push `main`.
- Do not commit `.env` or any real credentials.
- Do not reset away uncommitted work without reading and saving it first.
- Do not merge a branch that has failing tests.

---

## Development setup

See [README.md](README.md) for the full setup procedure.

---

## Documentation updates

When implementation changes an interface, schema, or behaviour:

1. Update `PROJECT_CONTEXT.md` (architecture and current state sections).
2. Append to `IMPLEMENTATION_LOG.md`.
3. Update `RECOVERY.md` (last known good state and next action).
4. Update `TASK_TRACKER.md` (task status).
5. Update the relevant `docs/` document if interfaces changed.

The four root-level continuity files must always agree with each other and with the actual code.

---

## Testing requirements

- All new features must have corresponding unit tests.
- Tests must use a temporary SQLite database, not the development database.
- Tests must mock the Groq API — do not require a real key for CI.
- A feature is only marked VERIFIED in TASK_TRACKER.md after a successful test run.

Run tests:

```bash
pytest tests/ -v
```

Run a specific file:

```bash
pytest tests/test_retrieval.py -v
```
