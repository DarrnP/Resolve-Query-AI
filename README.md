# Resolve Query AI

AI Copilot for Support Agents using FastAPI, Streamlit, LangChain agents, Groq LLMs, ChromaDB RAG, Mem0-style customer memory, SQLite, Docker, and EC2 deployment workflows.

This project is designed as a recruiter-friendly and interview-ready production AI application. It shows how a support team can use an AI copilot to create grounded, empathetic, and actionable customer reply drafts by combining ticket data, company knowledge, customer memory, and backend tool calls.


## Project Snapshot

| Area | Details |
|---|---|
| Domain | AI customer support automation |
| Primary user | Support agent handling customer tickets |
| Backend | FastAPI |
| Dashboard | Streamlit |
| LLM orchestration | LangChain agent with tool calling |
| LLM provider | Groq via `langchain-groq` |
| RAG store | ChromaDB persistent collection |
| Memory | Mem0 with Chroma vector store |
| Database | SQLite for customers, tickets, drafts |
| Documents | Banking policy and FAQ markdown files |
| Deployment | Docker, Docker Compose, GitHub Actions, EC2 guide |
| Main value | Faster support replies grounded in company policy, history, and tool data |

## What Problem This Solves

Support teams face three repeated production problems:

1. Agents spend too much time rewriting similar responses.
2. Answers become inconsistent when company policies are scattered across documents.
3. Support quality drops when agents do not know the customer's previous issues, plan, SLA, or ticket load.

This project solves those problems by building a support copilot that:

- Retrieves relevant policy from the knowledge base.
- Searches previous customer and company memories.
- Calls backend tools for structured support context.
- Generates a draft that a human agent can review before sending.
- Saves accepted resolutions as memory for future tickets.

## Core Features

- Create and list support tickets.
- Auto-generate support draft replies in the background.
- Manually regenerate drafts for a ticket.
- Accept or discard AI-generated drafts.
- Save accepted resolutions into long-term memory.
- Search customer memory from the dashboard.
- Ingest banking policy documents into ChromaDB.
- Display the exact context used for each draft.
- Expose FastAPI endpoints and a Streamlit support dashboard.
- Run locally with Docker Compose.
- Deploy simply to EC2 with GitHub Actions.






## Data Model

[Download Mermaid source](docs/diagrams/06_data_model.mmd)

```mermaid
erDiagram
    CUSTOMERS ||--o{ TICKETS : creates
    TICKETS ||--o{ DRAFTS : has

    CUSTOMERS {
        int id PK
        string email UK
        string name
        string company
        timestamp created_at
    }

    TICKETS {
        int id PK
        int customer_id FK
        string subject
        string description
        string status
        string priority
        timestamp created_at
        timestamp updated_at
    }

    DRAFTS {
        int id PK
        int ticket_id FK
        text content
        text context_used
        string status
        timestamp created_at
    }
```


## Knowledge Base

The project stores banking support documents in `knowledge_base/`:

- `saving-account-rule.md`
- `banking-kyc-and-account-update-rules.md`
- `banking-charges-and-minimum-balance.md`
- `banking-atm-cash-withdrawal-faq.md`

The ingestion pipeline:

[Download Mermaid source](docs/diagrams/05_knowledge_base_ingestion.mmd)

```mermaid
flowchart LR
    Docs[Markdown and Text Files] --> Split[Recursive Text Splitter]
    Split --> Chunks[Policy Chunks]
    Chunks --> Embed[Embedding Function]
    Embed --> Chroma[ChromaDB Collection]
    Chroma --> Search[Similarity Search at Draft Time]
```

## Memory Design

The system uses two memory scopes:

| Scope | Purpose |
|---|---|
| Customer memory | User-specific history, previous issues, preferences, accepted resolutions |
| Company memory | Shared account or organization-level issues affecting multiple users from the same company |

Why both scopes matter:

- A customer may have repeated account-specific issues.
- A company may have recurring integration, billing, SLA, or operational problems.
- Future support drafts become more personalized and less repetitive.
- Accepted resolutions become reusable support knowledge.

## Tool Calling

The support copilot has backend tools:

| Tool | Purpose |
|---|---|
| `lookup_customer_plan(customer_email)` | Returns plan tier, SLA hours, and priority queue status |
| `lookup_open_ticket_load(customer_email)` | Returns number of open tickets and customer load band |

Tool calling improves answer quality because the LLM does not guess structured data. It can use backend facts to create a more accurate support response.

## API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/health` | Health check |
| `POST` | `/api/tickets` | Create ticket and optionally auto-generate draft |
| `GET` | `/api/tickets` | List tickets |
| `GET` | `/api/tickets/{ticket_id}` | Get one ticket |
| `POST` | `/api/tickets/{ticket_id}/generate-draft` | Manually generate draft |
| `GET` | `/api/drafts/{ticket_id}` | Get latest draft for ticket |
| `PATCH` | `/api/drafts/{draft_id}` | Update draft content or status |
| `POST` | `/api/knowledge/ingest` | Ingest knowledge base into ChromaDB |
| `GET` | `/api/customers/{customer_id}/memories` | List customer memories |
| `GET` | `/api/customers/{customer_id}/memory-search` | Search customer and company memory |



## Typical Demo Flow

1. Start API and dashboard.
2. Click `Ingest Knowledge Base` in the dashboard.
3. Create a ticket with customer email, company, subject, description, and priority.
4. Let the system auto-generate the draft or click `Generate Draft`.
5. Expand `Context used` to show memory hits, KB hits, tool calls, and errors.
6. Edit the generated draft if needed.
7. Accept the draft.
8. Verify that the ticket becomes resolved and accepted resolution is saved into memory.
9. Use `Memory Probe` to search previous customer/company context.


## What I Would Improve Next

- Add JWT/OAuth authentication for support agents.
- Add RBAC for admin, agent, and reviewer roles.
- Replace SQLite with PostgreSQL for production.
- Add Redis caching for repeated memory, RAG, and draft requests.
- Add async job queue for background draft generation.
- Add prompt injection scanning and PII redaction.
- Add retrieval and draft quality evaluation.
- Add Langfuse, OpenTelemetry, or similar tracing.
- Add unit tests for repositories, services, routers, RAG ingestion, and tool calls.
- Add streaming draft generation in the dashboard.

## Repository Structure

```text
.
|-- app.py                                  # Streamlit dashboard
|-- main.py                                 # FastAPI entrypoint
|-- Dockerfile
|-- docker-compose.yml
|-- knowledge_base/                         # Banking policy docs
|-- docs/                                   # Deployment documentation
|-- tests/                                  # Tests
|-- customer_support_agent/
|   |-- api/
|   |   |-- app_factory.py                  # FastAPI app factory
|   |   |-- dependencies.py                 # Dependency injection
|   |   `-- routers/                        # API routes
|   |-- core/
|   |   `-- settings.py                     # Environment config
|   |-- integrations/
|   |   |-- rag/chroma_kb.py                # ChromaDB RAG
|   |   |-- memory/mem0_store.py            # Mem0 memory wrapper
|   |   `-- tools/support_tools.py          # Agent tools
|   |-- repositories/sqlite/                # SQLite repositories
|   |-- schemas/api.py                      # Pydantic models
|   `-- services/
|       |-- copilot_service.py              # Main AI copilot logic
|       |-- draft_service.py                # Draft persistence flow
|       `-- knowledge_service.py            # KB ingestion service
```


