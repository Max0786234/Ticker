# Database Schema — Ticker

> Last updated: 2026-10-10 (Phase 0 — Foundation)
> Update this document if the ORM model or seeding strategy changes.

---

## 1. Database technology

| Environment     | Database   | Driver                              |
|-----------------|------------|-------------------------------------|
| Local dev / MVP | SQLite     | Built-in (via SQLAlchemy)           |
| Optional deploy | PostgreSQL | `psycopg2` or `asyncpg` via NeonDB  |

The `DATABASE_URL` environment variable controls which backend is used.

SQLAlchemy abstracts the difference. The same models work with both databases.

---

## 2. HistoricalTicket model

This is the only table required for the MVP.

### SQLAlchemy ORM definition

```python
# app/db/models.py
from datetime import datetime
from sqlalchemy import Column, Integer, String, Text, DateTime
from app.db.database import Base

class HistoricalTicket(Base):
    __tablename__ = "historical_tickets"

    id          = Column(Integer, primary_key=True, index=True)
    title       = Column(String(200), nullable=False)
    description = Column(Text, nullable=False)
    category    = Column(String(100), nullable=False)
    resolution  = Column(Text, nullable=False)
    status      = Column(String(50), nullable=False, default="resolved")
    created_at  = Column(DateTime, default=datetime.utcnow)
```

### Column reference

| Column      | SQLAlchemy type | Max length | Nullable | Default         | Description                              |
|-------------|-----------------|------------|----------|-----------------|------------------------------------------|
| `id`        | Integer         | —          | No       | Auto-increment  | Primary key                              |
| `title`     | String          | 200        | No       | —               | Short summary of the IT issue            |
| `description` | Text          | Unlimited  | No       | —               | Full problem description                 |
| `category`  | String          | 100        | No       | —               | Issue category (e.g. "VPN", "Email")     |
| `resolution` | Text           | Unlimited  | No       | —               | Steps taken to resolve the issue         |
| `status`    | String          | 50         | No       | `"resolved"`    | Ticket lifecycle status                  |
| `created_at` | DateTime       | —          | No       | `datetime.utcnow` | When the ticket was created            |

---

## 3. Category taxonomy for synthetic data

The seed dataset covers these categories:

| Category             | Example issues                                        |
|----------------------|-------------------------------------------------------|
| VPN                  | Client fails, disconnects, credentials expired        |
| Network / Wi-Fi      | No internet, slow connection, DHCP failure            |
| Password / Auth      | Locked accounts, MFA failures, expired passwords      |
| Email / Outlook      | Crashes, send/receive failures, calendar sync         |
| Printer              | Offline, driver missing, paper jam, network printer   |
| Software Install     | Permission denied, package corruption, activation     |
| Permissions / Access | Shared drive denied, admin rights, file access        |
| Performance          | Slow startup, high CPU, memory exhaustion             |
| OS / System Errors   | Blue screen, disk errors, update failures             |
| Audio / Video        | No sound in meetings, webcam not detected, Teams issues |

---

## 4. Seeding strategy

### Source file

`data/synthetic_tickets.json` — approximately 50 records as a JSON array.

```json
[
  {
    "title": "VPN disconnects repeatedly after Windows update",
    "description": "After installing the latest Windows security update, the corporate VPN client disconnects every 10–15 minutes and requires manual reconnection.",
    "category": "VPN",
    "resolution": "1. Rolled back the specific KB update causing the conflict. 2. Updated the VPN client to the latest version. 3. Disabled IPv6 on the network adapter as a workaround. 4. Re-registered the VPN adapter in Device Manager.",
    "status": "resolved"
  }
]
```

### Idempotency mechanism

The seeder checks the current ticket count before inserting:

```python
# app/db/seed.py  (pseudocode — actual implementation in Phase 1)
existing = session.query(HistoricalTicket).count()
if existing >= len(seed_data):
    print(f"Database already seeded ({existing} tickets). Skipping.")
    return
```

Running `python -m app.db.seed` multiple times is safe. It will not duplicate data.

### Running the seeder

```bash
python -m app.db.seed
```

Expected output on first run:
```
Seeding database with 50 historical tickets...
Done. 50 tickets inserted.
```

Expected output on subsequent runs:
```
Database already seeded (50 tickets). Skipping.
```

---

## 5. Text used for TF-IDF retrieval

The retrieval service combines these columns into a single searchable string per ticket:

```python
searchable_text = f"{ticket.title} {ticket.description} {ticket.category}"
```

The `resolution` column is NOT included in the searchable text — it is only used as evidence for the LLM after matching. This is intentional: we want to match on the *problem*, not the *solution*.

---

## 6. Session management

```python
# app/db/database.py  (pseudocode)
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

class Base(DeclarativeBase):
    pass

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

`connect_args={"check_same_thread": False}` is required for SQLite when used with FastAPI's async thread pool. This is not needed for PostgreSQL.

---

## 7. Test database isolation

Tests must NOT use `ticker.db`. Use an isolated, temporary database:

```python
# tests/conftest.py  (pseudocode)
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.db.database import Base

@pytest.fixture(scope="session")
def test_engine():
    engine = create_engine("sqlite:///./test_ticker.db")
    Base.metadata.create_all(engine)
    yield engine
    Base.metadata.drop_all(engine)
    engine.dispose()
```

Alternatively, use `sqlite:///:memory:` for an in-memory database that is destroyed after each test session.

---

## 8. Future: PostgreSQL/NeonDB migration

To switch to PostgreSQL:

1. Set `DATABASE_URL=postgresql+psycopg2://user:pass@host/db` in `.env`.
2. Install: `pip install psycopg2-binary` (or `asyncpg` for async).
3. Remove `connect_args={"check_same_thread": False}` from the engine call.
4. Run `python -m app.db.seed` to populate the new database.

No model changes are required. SQLAlchemy handles the dialect difference.

Consider Alembic for migration management if the schema evolves before the PostgreSQL switch.
