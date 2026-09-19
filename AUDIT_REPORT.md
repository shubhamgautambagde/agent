# Auto Voice Calling Agent: Comprehensive Codebase Audit Report (Phase 1)

*Date: 2026-09-19 | Target: Transformation into a Production-Ready Auto Voice Calling Agent Platform*

---

## 1. Repository & Technology Inventory

| Layer / Subsystem | Current Technology & Implementation Details | Status |
| :--- | :--- | :--- |
| **Backend Framework** | **Go 1.25** microservices (`assistant-api`, `web-api`, `integration-api`, `endpoint-api`), `cmux` single-port multiplexer, Gin REST (`/v1`), binary gRPC (`pkg/clients/`), Zap structured logger. | **READY** |
| **Frontend Framework** | **React 18.2**, TypeScript, Tailwind CSS (v4 CLI), Craco, Zustand stores, Carbon design tokens, React Router v6. | **READY** |
| **Document / RAG Engine**| **Python 3.11** FastAPI + Celery workers (`document-api`), unstructured PDF/doc extractors. | **READY** |
| **Relational Database** | **PostgreSQL 15** with GORM ORM, 60 sequential migration scripts in `assistant-api`, 13 in `web-api`. | **READY** |
| **Cache & PubSub** | **Redis 7** used for GORM L2 query caching, distributed locking, and event publication. | **READY** |
| **Vector Engine** | **OpenSearch 2.11** with tenant-scoped vector indices and BM25 hybrid search. | **READY** |
| **Authentication & RBAC** | JWT principle resolution, password hashing (bcrypt), token verification, Org/Project role schemas (`web-api`). | **PARTIAL** (Needs explicit Campaign & Lead roles) |
| **Telephony Channels** | Pluggable channel framework: `Twilio`, `Telnyx`, `Vonage`, `Exotel`, `Asterisk`, `Vobiz`, raw `SIP`. | **READY** |
| **SIP Server** | Dedicated SIP listener on port `4573` (`assistant-api/sip`), SIP INVITE, BYE, CANCEL, REFER (transfer) handling. | **READY** |
| **Outbound Calling Engine**| `OutboundDispatcher` in `assistant-api` capable of dialing via Twilio/Telnyx/SIP to an external phone number. | **PARTIAL** (Ad-hoc single dispatch; missing campaign automated queue) |
| **STT Transformers** | `Deepgram`, `Whisper`, `Google`, `Azure`, `AssemblyAI`, `Speechmatics`, `Sarvam` (Hindi/Marathi), `Minimax`, `Groq`. | **READY** |
| **TTS Transformers** | `ElevenLabs`, `Cartesia`, `OpenAI Audio`, `Azure`, `PlayHT`, `Sarvam` (Indic voices), `Smallest`, `Neuphonic`. | **READY** |
| **LLM Gateway** | `OpenAI`, `Anthropic Claude`, `Google Gemini`, `Groq`, `Azure OpenAI`, `Cohere`, `VertexAI`, `xAI`, `Custom LLM`. | **READY** |
| **VAD & Interruption** | `Silero VAD`, `Ten VAD`, `FireRed VAD`, energy thresholding, barge-in stream truncation. | **READY** |
| **Human Transfer** | Local tool `transferCallCaller` and SIP REFER signaling (`sip_pipeline.TransferInitiatedPipeline`). | **READY** |
| **Conversation State** | Real-time `Communication` state machine tracking user packets, agent response tokens, and event metrics. | **READY** |
| **Transcripts & Audio** | Persistent storage of messages, audio packet URLs, timestamps, latency metrics in `conversations`. | **READY** |
| **Lead CRM System** | Lead entity, contacts, phone validation, DND status, consent tracking, CSV/XLSX import. | **MISSING** |
| **Campaign Manager** | Campaign entity, scheduling, business hours, retry delay, concurrent calling limits, batch execution. | **MISSING** |
| **Outbound Calling Queue** | Automated background worker to poll eligible campaign leads, enforce concurrency, and initiate calls. | **MISSING** |
| **AI Summary & Scoring** | Post-call LLM prompt execution to generate qualification score (0-100), intent, budget, and summary. | **PARTIAL** (Analysis endpoint exists; needs structured auto-scoring) |
| **Appointments & Follow-ups**| Dedicated calendar booking models, follow-up scheduler, reminders table. | **MISSING** |
| **Live Call Monitor UI** | Real-time live call active board with duration, status, and live transcript view. | **PARTIAL** (Observability exists; needs dedicated campaign call board) |

---

## 2. Component Categorization Matrix

### 🟢 READY (Reuse As-Is)
1. **Core Voice Streaming Pipeline**: `assistant-api` WebSocket, WebRTC, and gRPC bidirectional streamers.
2. **STT/TTS Provider Matrix**: 20+ transformers already implemented with standard packet formats.
3. **Telephony Provider Abstractions**: Twilio, Telnyx, Vonage, Exotel, Asterisk, and SIP bridges.
4. **VAD & Barge-In**: Real-time silence and speech detection (`silero_vad`).
5. **LLM Provider Gateway**: `integration-api` unified caller for OpenAI, Anthropic, Gemini, Groq.
6. **Tool Calling & Execution**: `endpoint-api` dynamic function runner with rate limiting.
7. **Identity Core**: User authentication, password hashing, and encrypted credential Vaults in `web-api`.

### 🟡 PARTIAL (Extend / Bridge)
1. **Outbound Dispatcher**: Currently triggers single calls via context; needs integration with a batch campaign queue.
2. **Post-Call Analysis**: `assistant-api/internal/analysis` can trigger webhooks/endpoints; needs dedicated structured AI summary & lead scoring extractor.
3. **RBAC**: User roles exist at Org/Project level; needs specific role definitions: *Super Admin, Admin, Manager, Voice Agent Operator, Viewer*.
4. **Observability UI**: Basic traces exist; needs dedicated Auto-Calling Campaign Dashboard and Live Monitor.

### 🔴 MISSING (Must Be Developed)
1. **Lead Management (CRM)**:
   - Database schema: `leads` with 14 lifecycle statuses (`NEW`, `QUEUED`, `CALLING`, `CONNECTED`, `NO_ANSWER`, `BUSY`, `CALLBACK`, `INTERESTED`, `QUALIFIED`, `NOT_INTERESTED`, `APPOINTMENT`, `CONVERTED`, `LOST`, `DO_NOT_CALL`).
   - CSV / Excel lead importer with duplicate detection, phone normalization, and column mapping.
2. **Campaign Management**:
   - Database schema: `campaigns` & `campaign_leads`.
   - Campaign settings: Calling schedule, business hours, max attempts, retry delay, concurrency limits, daily caps.
   - Campaign controls: Start, Pause, Resume, Stop, Retry Failed.
3. **Outbound Calling Queue & Background Worker**:
   - Concurrency controller and queue dispatcher that checks calling hours, DND list, and consent before dialing.
4. **Appointments & Follow-ups**:
   - Database schema: `appointments` & `follow_ups`.
   - Tool calling integration for AI to book appointments and schedule callbacks during conversation.
5. **Compliance & Safety Engine**:
   - Automated DND check, instant opt-out detection ("stop calling" -> auto mark `DO_NOT_CALL`), and calling-hour enforcement.
6. **Frontend Auto-Calling Center**:
   - Leads CRM view (table, filter by status/tag, CSV upload modal).
   - Campaigns manager (create wizard, lead assignment, schedule configuration).
   - Live Call Monitoring & Analytics view.
   - Appointments & Follow-ups calendar/table.

---

## 3. Implementation Roadmap (Phases 2-20)

```
Phase 1: Codebase Audit [COMPLETE]
  ├── Phase 2: Architecture & Schema Alignment (Leads, Campaigns, Appointments, Follow-ups)
  ├── Phase 3: Database Migrations & Entities
  ├── Phase 4: RBAC & Permission Enforcement
  ├── Phase 5: Lead CRM & CSV/XLSX Import Engine
  ├── Phase 6: Campaign Management Engine
  ├── Phase 7: AI Voice Agent Configuration Extensions (Hindi/Marathi/English presets)
  ├── Phase 8: Telephony & Outbound Dispatcher Bridge
  ├── Phase 9: Background Calling Queue & Concurrency Worker
  ├── Phase 10: Conversation Engine Compliance & Opt-Out Interceptor
  ├── Phase 11: Call Recording & Transcript Linkage
  ├── Phase 12: Post-Call AI Summary & Configurable Lead Scoring (0-100)
  ├── Phase 13: Appointment Booking & Follow-up Scheduler Tools
  ├── Phase 14: Human Transfer Bridge Validation
  ├── Phase 15: Frontend Auto-Calling Center (Leads, Campaigns, Live Monitor, Analytics)
  ├── Phase 16: Compliance & Safety Hardening (DND, Calling Hours)
  ├── Phase 17: Security & Secret Isolation
  ├── Phase 18: Unit, Integration & E2E Verification
  ├── Phase 19: Docker & Environment Hardening
  └── Phase 20: Production Audit & Verification
```
