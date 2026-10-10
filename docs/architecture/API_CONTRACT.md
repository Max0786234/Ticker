# API Contract — Ticker

> Last updated: 2026-10-10 (Phase 0 — Foundation)
> This document is the shared contract between the FastAPI backend and the Streamlit frontend.
> Any change to a request or response schema must update this document first.

---

## Base URL

```
http://localhost:8000
```

Interactive documentation (Swagger UI): **http://localhost:8000/docs**

---

## Authentication

No authentication is required for the MVP. All endpoints are public.

---

## Common response headers

```
Content-Type: application/json
```

---

## Endpoints

---

### GET /health

Returns the operational status of the application.

**Request:** No body, no parameters.

**Response — HTTP 200:**

```json
{
  "status": "ok",
  "version": "0.1.0"
}
```

**Pydantic schema:**

```python
class HealthResponse(BaseModel):
    status: str
    version: str
```

---

### GET /tickets

Returns historical tickets from the database.

**Query parameters:**

| Parameter | Type    | Default | Description                              |
|-----------|---------|---------|------------------------------------------|
| `limit`   | integer | 100     | Maximum number of tickets to return      |
| `offset`  | integer | 0       | Number of tickets to skip (for pagination)|

**Response — HTTP 200:**

```json
{
  "tickets": [
    {
      "id": 1,
      "title": "VPN disconnects after Windows update",
      "description": "...",
      "category": "VPN",
      "resolution": "...",
      "status": "resolved",
      "created_at": "2024-01-15T10:30:00"
    }
  ],
  "total": 50
}
```

**Pydantic schemas:**

```python
class TicketResponse(BaseModel):
    id: int
    title: str
    description: str
    category: str
    resolution: str
    status: str
    created_at: datetime

class TicketsListResponse(BaseModel):
    tickets: List[TicketResponse]
    total: int
```

**Error responses:**

| HTTP Status | When                                    |
|-------------|------------------------------------------|
| 422         | Invalid query parameter types           |

---

### POST /tickets

Creates and persists a new historical ticket.

**Request body:**

```json
{
  "title": "Outlook crashes on startup",
  "description": "Microsoft Outlook crashes immediately after launch on Windows 11.",
  "category": "Email",
  "resolution": "Repaired the Office installation via Programs and Features.",
  "status": "resolved"
}
```

**Pydantic schema:**

```python
class TicketCreate(BaseModel):
    title: str = Field(..., min_length=3, max_length=200)
    description: str = Field(..., min_length=10)
    category: str = Field(..., min_length=2, max_length=100)
    resolution: str = Field(..., min_length=10)
    status: str = Field(default="resolved", max_length=50)
```

**Response — HTTP 201:**

```json
{
  "id": 51,
  "title": "Outlook crashes on startup",
  "description": "Microsoft Outlook crashes immediately after launch on Windows 11.",
  "category": "Email",
  "resolution": "Repaired the Office installation via Programs and Features.",
  "status": "resolved",
  "created_at": "2026-10-10T19:00:00"
}
```

**Error responses:**

| HTTP Status | When                                           |
|-------------|------------------------------------------------|
| 422         | Missing required fields or validation failure  |

---

### POST /resolve

**This is the primary endpoint.** Accepts a user's IT issue and returns five actionable resolution steps based on historical ticket evidence.

**Request body:**

```json
{
  "issue_description": "My laptop connects to office Wi-Fi but the company VPN keeps failing. It was working yesterday."
}
```

**Pydantic schema:**

```python
class ResolveRequest(BaseModel):
    issue_description: str = Field(
        ...,
        min_length=10,
        max_length=2000,
        description="The user's IT issue in plain English"
    )
```

**Validation rules:**
- `issue_description` must be between 10 and 2000 characters.
- Empty string or whitespace-only strings are rejected (422).

---

**Success response — HTTP 200, `generation_status: "success"`:**

```json
{
  "issue_description": "My laptop connects to office Wi-Fi but the company VPN keeps failing.",
  "resolution_steps": [
    "Step 1: Check that the VPN client software is up to date by opening the client and navigating to Help > Check for Updates.",
    "Step 2: Restart the VPN client completely — quit the application, wait 10 seconds, then relaunch it.",
    "Step 3: Verify your VPN credentials have not expired by attempting to log in to the web portal at vpn.company.com.",
    "Step 4: Flush the DNS cache by opening Command Prompt as Administrator and running: ipconfig /flushdns",
    "Step 5: If the issue persists, uninstall and reinstall the VPN client, then re-import the configuration profile from IT."
  ],
  "matched_tickets": [
    {
      "ticket_id": 3,
      "title": "VPN client fails to connect after OS update",
      "category": "VPN",
      "similarity_score": 0.68
    },
    {
      "ticket_id": 11,
      "title": "Cannot connect to corporate VPN from home office",
      "category": "VPN",
      "similarity_score": 0.45
    }
  ],
  "generation_status": "success",
  "message": null
}
```

---

**Insufficient evidence response — HTTP 200, `generation_status: "insufficient_evidence"`:**

Returned when no historical tickets meet the similarity threshold. The LLM is NOT called in this case.

```json
{
  "issue_description": "The office coffee machine display shows E7.",
  "resolution_steps": [],
  "matched_tickets": [],
  "generation_status": "insufficient_evidence",
  "message": "No historical tickets were similar enough to your issue. Please contact the IT helpdesk directly."
}
```

---

**Generation error response — HTTP 200, `generation_status: "generation_error"`:**

Returned when the Groq API fails or returns an invalid format after one retry.

```json
{
  "issue_description": "My laptop connects to office Wi-Fi but the VPN fails.",
  "resolution_steps": [],
  "matched_tickets": [
    {
      "ticket_id": 3,
      "title": "VPN client fails to connect after OS update",
      "category": "VPN",
      "similarity_score": 0.68
    }
  ],
  "generation_status": "generation_error",
  "message": "AI generation failed. Relevant historical tickets were found but could not be synthesised. Please try again or contact the IT helpdesk."
}
```

**Note:** `matched_tickets` may be populated even on `generation_error` — the retrieval succeeded; only the LLM step failed.

---

**Pydantic response schemas:**

```python
class MatchedTicket(BaseModel):
    ticket_id: int
    title: str
    category: str
    similarity_score: float

class ResolveResponse(BaseModel):
    issue_description: str
    resolution_steps: List[str]
    matched_tickets: List[MatchedTicket]
    generation_status: Literal["success", "insufficient_evidence", "generation_error"]
    message: Optional[str]
```

**Error responses:**

| HTTP Status | When                                                    |
|-------------|---------------------------------------------------------|
| 422         | Request body fails validation (too short, too long)     |
| 500         | Unexpected server error (should not occur in normal operation) |

---

## generation_status reference

| Value                   | Meaning                                                                      | `resolution_steps` | `matched_tickets` |
|-------------------------|------------------------------------------------------------------------------|--------------------|-------------------|
| `success`               | 5 steps generated and validated                                              | Exactly 5 strings  | 1–5 tickets       |
| `insufficient_evidence` | No tickets above similarity threshold; LLM was NOT called                   | Empty list `[]`    | Empty list `[]`   |
| `generation_error`      | Groq API error or invalid format after one retry; LLM was called but failed  | Empty list `[]`    | May be non-empty  |

---

## Field constraints summary

| Field                       | Type    | Min     | Max       | Notes                    |
|-----------------------------|---------|---------|-----------|--------------------------|
| `issue_description`         | string  | 10 chars| 2000 chars| Whitespace-only rejected |
| `TicketCreate.title`        | string  | 3 chars | 200 chars |                          |
| `TicketCreate.description`  | string  | 10 chars| —         |                          |
| `TicketCreate.category`     | string  | 2 chars | 100 chars |                          |
| `TicketCreate.resolution`   | string  | 10 chars| —         |                          |
| `similarity_score`          | float   | 0.0     | 1.0       | Cosine similarity score  |
| `resolution_steps` (success)| list    | 5 items | 5 items   | Exactly 5 on success     |

---

## Example curl commands

```bash
# Health check
curl http://localhost:8000/health

# List tickets (first 10)
curl "http://localhost:8000/tickets?limit=10&offset=0"

# Create a ticket
curl -X POST http://localhost:8000/tickets \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Outlook crashes on startup",
    "description": "Outlook crashes immediately after launch on Windows 11.",
    "category": "Email",
    "resolution": "Repaired Office installation via Programs and Features.",
    "status": "resolved"
  }'

# Resolve an issue
curl -X POST http://localhost:8000/resolve \
  -H "Content-Type: application/json" \
  -d '{
    "issue_description": "My VPN keeps disconnecting every few minutes when working from home."
  }'
```
