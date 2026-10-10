# RECOVERY.md — Ticker

> **This file is for immediate use when a session is interrupted, crashed, or taken over by a different agent.**
>
> ⚠️ **Do not assume the previous AI completed its last task.**
> Inspect the repository, Git status, relevant files, and test results before continuing.
> Never overwrite working code without understanding its current state.

---

## Current milestone

**Phase 0 — Project Foundation**

---

## Last known working state

The repository was cloned from GitHub (empty). All foundation files are being created and committed to the `chore/project-foundation` branch.

No application code exists yet. No tests exist yet. No database exists yet.

---

## Last completed task

Creating foundation documentation files and directory structure. Foundation commit to `main` and `chore/project-foundation` branch.

---

## Interrupted or in-progress task

None (as of this writing). If you find an interrupted task, check `git status` and `git stash list`.

---

## Files potentially modified but not verified

At the end of Phase 0, all files in the list below were created. They have not been run through the application yet because no application code exists.

- `README.md` — rendered correctly in terminal but not verified on GitHub.
- `requirements.txt` — not yet tested in a clean virtual environment.
- `docs/architecture/*.md` — correct as documentation; not linked by running code.
- All `__init__.py` stubs — present but no imports tested.

---

## Last commands and their results

```bash
# Cloning
git clone https://github.com/Max0786234/Ticker.git <workspace>
→ Succeeded (empty repository warning — expected)

# Verifying identity
git config user.name   →  Max
git config user.email  →  max23314@iiitd.ac.in
```

Check `IMPLEMENTATION_LOG.md` Entry 001 for the complete record.

---

## Known blockers

None during Phase 0.

For Phase 1, the developer will need a Groq API key in `.env` before live LLM tests. Unit tests mock the Groq API and do not need the key.

---

## Exact next safe action

1. Confirm Git status is clean on `chore/project-foundation`:
   ```bash
   git -C "c:\Users\max03\Downloads\WorkM\Ticker" status
   git -C "c:\Users\max03\Downloads\WorkM\Ticker" log --oneline -5
   ```

2. Confirm the foundation branch was pushed:
   ```bash
   git -C "c:\Users\max03\Downloads\WorkM\Ticker" branch -a
   ```

3. If the push succeeded and status is clean, begin **Phase 1**:
   ```bash
   git -C "c:\Users\max03\Downloads\WorkM\Ticker" checkout main
   git -C "c:\Users\max03\Downloads\WorkM\Ticker" checkout -b feature/database-seed
   ```

4. Read `PROJECT_CONTEXT.md` Section 14 ("Next safe task") for Phase 1 implementation steps.

---

## Verification commands

```bash
# Confirm repository state
git -C "c:\Users\max03\Downloads\WorkM\Ticker" log --oneline
git -C "c:\Users\max03\Downloads\WorkM\Ticker" branch -a
git -C "c:\Users\max03\Downloads\WorkM\Ticker" status

# Confirm no secrets were committed
git -C "c:\Users\max03\Downloads\WorkM\Ticker" grep -r "GROQ_API_KEY" -- "*.py" "*.json"

# Confirm .env is not tracked
git -C "c:\Users\max03\Downloads\WorkM\Ticker" ls-files --error-unmatch .env

# List all tracked files
git -C "c:\Users\max03\Downloads\WorkM\Ticker" ls-files
```

---

## Recovery quick-reference for Phase 1

If the session was interrupted during Phase 1 (database and seed):

1. Check if `app/db/models.py` exists and contains the `HistoricalTicket` class.
2. Check if `data/synthetic_tickets.json` exists and has ~50 records.
3. Check if `app/db/seed.py` exists.
4. Run `pytest tests/test_seed.py -v` to see if seed tests pass.
5. If files exist but tests fail, read the error message before making changes.
6. If files are missing, re-implement from `PROJECT_CONTEXT.md` Section 7 (Database Schema) and Section 14 (Next safe task).
