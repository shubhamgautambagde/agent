# Persistent Project Memory & Context Ledger (Memory.md)

*Purpose: This file tracks active codebase state, recent architectural decisions, verified commands, and ongoing workstreams so any AI assistant or developer can resume work without re-reading the entire repository.*

---

## 1. Project Snapshot

- **Project**: Rapida Voice AI (`github.com/rapidaai`)
- **Type**: Distributed real-time voice AI orchestration platform
- **Languages**: Go 1.25, TypeScript / React 18, Python 3.11 (FastAPI)
- **Primary Transport**: Bidirectional gRPC, WebRTC, SIP (Port 4573), Gin REST (cmux multiplexed)
- **Data Stores**: PostgreSQL 15 (`5432`), Redis 7 (`6379`), OpenSearch 2.11 (`9200`), Nginx (`8080`)

---

## 2. Port & Service Allocation Map

| Service | Port | Protocol | Entry Point / Path | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **`web-api`** | `9001` | gRPC / REST | `cmd/web/web.go` | Auth, orgs, projects, credential vaults |
| **`assistant-api`** | `9007`<br>`4573` | gRPC / REST<br>SIP Audio | `cmd/assistant/assistant.go` | Voice loop, STT/TTS transformers, VAD |
| **`integration-api`** | `9004` | gRPC | `cmd/integration/integration.go` | LLM connectors (OpenAI, Anthropic, Gemini, Groq) |
| **`endpoint-api`** | `9005` | gRPC / REST | `cmd/endpoint/endpoint.go` | Dynamic tool caller, webhooks, rate limits |
| **`document-api`** | `9010` | REST / Celery | `api/document-api/app/main.py` | Document ingestion, chunking, embeddings |
| **`ui`** | `3000` | HTTP / WS | `ui/src/index.tsx` | React 18, Tailwind v4, Craco dev server |

---

## 3. Key Technical Decisions & Patterns

1. **Protocol Multiplexing (`cmux`)**:
   - Each Go service runs on a single TCP port, multiplexing HTTP/2 (gRPC), grpc-web, and HTTP/1.1 (Gin).
2. **Inter-Service Communication**:
   - All Go service-to-service calls use typed gRPC stubs in `pkg/clients/`. Direct HTTP between services is forbidden.
3. **Database Entities**:
   - All relational tables compose `Audited` (Snowflake ID generated on `BeforeCreate`), `Mutable`, and `Organizational`.
4. **STT & TTS Transformer Pattern**:
   - Providers implement `Transformers[AudioPacket]` and `Transformers[TextPacket]` in `api/assistant-api/internal/transformer/`.
5. **Forbidden Patterns Enforced**:
   - The stem `normaliz*` is banned across the codebase.
   - Generic `util` / `helper` packages are banned.
   - Redux is banned in frontend; Zustand stores (`use-*-page-store.ts`) are used.

---

## 4. Key Development & Verification Commands

```bash
# Infrastructure
just deps                  # Start PostgreSQL and Redis containers
just up-all                # Start full Docker Compose stack

# Local Go Services (without Docker)
go run cmd/web/web.go
go run cmd/assistant/assistant.go
go run cmd/integration/integration.go
go run cmd/endpoint/endpoint.go

# Local Python Document Service
PYTHONPATH=api/document-api uvicorn app.main:app --host 0.0.0.0 --port 9010

# Local UI
cd ui && yarn start:dev    # Tailwind watch + Craco dev server

# Testing & Verification
go test ./...              # All Go tests
golangci-lint run          # Go linter
cd ui && yarn test         # UI Jest tests
cd ui && yarn checkTs      # UI TypeScript check
cd ui && yarn lint         # UI ESLint
```

---

## 5. Specification Documents Map

- [`AUDIT_REPORT.md`](./AUDIT_REPORT.md): Comprehensive Phase 1 audit report categorizing READY, PARTIAL, and MISSING modules.
- [`PRD.md`](./PRD.md): Product requirements, personas, capabilities, and non-functional goals.
- [`Architecture.md`](./Architecture.md): Detailed component architecture, data flow diagrams, and directory tree.
- [`Rules.md`](./Rules.md): Coding rules, strict boundaries, and AI governance.
- [`Phases.md`](./Phases.md): Milestone roadmap across development phases.
- [`Design.md`](./Design.md): UI styling tokens, color modes, typography, and visualizers.
- [`Memory.md`](./Memory.md): *This document* — active context and progress ledger.

---

## 6. Auto Voice Calling Agent Extensions (Completed)

1. **Audit & Roadmap**:
   - Completed Phase 1 codebase audit recorded in [`AUDIT_REPORT.md`](./AUDIT_REPORT.md).
2. **Database & Migrations**:
   - Added migrations `000061`, `000062`, `000063` for `leads`, `campaigns`, `campaign_leads`, `appointments`, and `follow_ups`.
   - Created typed GORM models in `api/assistant-api/internal/entity/` (`leads`, `campaigns`, `appointments`, `followups`).
3. **Core Services**:
   - `LeadService`: CSV batch import, E.164 phone canonicalization, duplicate rejection, and instant DND opt-out suppression.
   - `CampaignService`: Concurrency control, calling hours restriction, lead assignment, retry failed calls, and real-time statistics.
   - `CallingQueueWorker`: Background worker checking active campaigns, time-window compliance, concurrency limits, and dispatching calls via `OutboundDispatcher`.
   - `CallAnalysisService`: Post-call NLP/rule evaluation producing 0-100 lead score, HOT/WARM/MEDIUM/COLD classification, and requirement summaries.
4. **Conversational In-Call Tools**:
   - `book_appointment_caller`: Enables voice AI to confirm and persist calendar bookings.
   - `schedule_followup_caller`: Schedules callback requests with specific date/time.
   - `opt_out_caller`: Intercepts customer opt-out ("stop calling me"), marks DND, and gracefully ends conversation.
5. **Role-Based Access Control**:
   - Created `pkg/types/rbac/roles.go` supporting `SUPER_ADMIN`, `ADMIN`, `MANAGER`, `OPERATOR`, and `VIEWER`.
6. **Frontend Auto-Calling Center**:
   - `CampaignsPage` (`/campaigns`): Live cards, start/pause/stop controls, creation modal.
   - `LeadsPage` (`/leads`): CRM table, search, status filters, score badges, CSV upload modal.
   - `CallsPage` (`/calls`): Live call monitoring, duration counter, streaming transcript view.
   - `AppointmentsPage` (`/appointments`): Scheduled demos and callback tracking.
   - Navigation: Wired into `ui/src/app/routes/` and `SidebarNavigation`.
7. **Environment & Secrets**:
   - Documented complete `.env.example` with telephony, STT/TTS, LLM, and database keys.

