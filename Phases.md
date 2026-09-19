# Project Roadmap & Phases (Phases.md)

This document breaks down the development, hardening, and expansion of **Rapida Voice AI** into sequential, manageable phases.

---

## Roadmap Overview

```
Phase 1: Foundation & Microservices Baseline
   └── Phase 2: Real-Time Audio Engine & Streaming
          └── Phase 3: Telephony & SIP Transports
                 └── Phase 4: LLM Gateway & Tool Execution
                        └── Phase 5: Knowledge & Hybrid RAG
                               └── Phase 6: Console UI & Observability
```

---

## Phase 1: Foundation & Microservices Baseline
*Goal*: Establish reliable service bootstrapping, shared infrastructure, and tenant boundaries.

- [x] **Infrastructure Orchestration**: Docker Compose stack (`docker-compose.yml`) for PostgreSQL 15, Redis 7, OpenSearch 2.11, and Nginx.
- [x] **Common Runtime Utilities**: Shared logging (Zap SugaredLogger), Snowflake ID generation, and configuration loading via Viper (`config/config.go`).
- [x] **Schema Migrations**: Database schemas and migration pipelines for `web-api`, `assistant-api`, and `endpoint-api`.
- [ ] **Multi-Tenant Identity & Vaults**: Secure tenant scoping (`organization_id`, `project_id`) and AES-256 GCM encrypted credential vaults for third-party API keys.
- [ ] **Automated CI Validation**: Pre-commit hooks, linting (`golangci-lint`), and TypeScript type checking.

---

## Phase 2: Real-Time Audio Engine & Streaming
*Goal*: Deliver low-latency bidirectional voice conversations with accurate barge-in detection.

- [x] **Bidirectional gRPC Audio Protocol**: Protobuf contracts for bidirectional streaming of raw audio frames between clients and `assistant-api`.
- [x] **STT Transformers**: Deepgram streaming transformer and OpenAI Whisper batch/stream integration.
- [x] **TTS Transformers**: ElevenLabs streaming TTS and Cartesia ultra-low latency integration.
- [ ] **VAD & Interruption Calibration**: In-stream Voice Activity Detection (Silero VAD) to immediately pause TTS output upon user interruption.
- [ ] **WebRTC Ingress**: Direct browser-to-server WebRTC channel with OPUS audio encoding and echo cancellation.

---

## Phase 3: Telephony & SIP Transports
*Goal*: Enable inbound and outbound phone calling via carrier SIP trunks and telephony providers.

- [ ] **SIP Audio Server**: Dedicated SIP listener on port `4573` within `assistant-api`.
- [ ] **Telephony Provider Connectors**: Webhook and media stream handlers for Twilio, Telnyx, and Vonage.
- [ ] **Call Lifecycle Management**: Call initiation, transfer, DTMF tone detection, recording, and graceful hangup.
- [ ] **Audio Resampling & Jitter Buffering**: 8kHz telephony PCMU/PCMA to 16kHz/24kHz pipeline transcoding.

---

## Phase 4: LLM Gateway & Tool Execution
*Goal*: Pluggable model providers and high-reliability dynamic tool execution.

- [x] **Provider Multiplexing (`integration-api`)**: Unified interfaces for OpenAI, Anthropic Claude, Google Gemini, and Groq.
- [x] **Streaming Token Buffer**: Token-to-sentence chunking buffer to stream full syntactic phrases into TTS engines without awkward pauses.
- [ ] **Dynamic Tool Execution (`endpoint-api`)**: External webhook function calling with configurable timeouts, retries, and rate limits.
- [ ] **Automated Provider Failover**: Seamless failover to secondary LLM models when the primary upstream provider experiences rate-limiting or latency spikes.

---

## Phase 5: Knowledge & Hybrid RAG Engine
*Goal*: Accurate, tenant-isolated knowledge retrieval during live voice calls.

- [x] **Document Ingestion Service (`document-api`)**: Python FastAPI + Celery workers for PDF, DOCX, and Markdown parsing.
- [x] **Vector Embedding Pipeline**: Integration with embedding models to generate document chunk vectors.
- [ ] **Tenant-Isolated OpenSearch Storage**: OpenSearch index management with strict tenant separation and BM25 + kNN hybrid search.
- [ ] **Voice Prompt Context Injection**: Sub-50ms query execution and dynamic injection of retrieved knowledge snippets into LLM system prompts.

---

## Phase 6: Console UI, Observability & Analytics
*Goal*: Enterprise-grade operator dashboard and agency white-labeling capabilities.

- [x] **React Dashboard (`ui`)**: React 18, TypeScript, and Tailwind CSS management console.
- [x] **Assistant Studio**: Configuration interface for voice parameters, system prompts, tools, and provider selection.
- [ ] **Live Call Debugger & Audio Playback**: Real-time waveform visualizer, call transcript timeline, and recorded session playback.
- [ ] **Latency Waterfall Analytics**: Step-by-step telemetry measuring TTFA (Time-to-First-Audio), STT latency, LLM first-token time, and TTS synthesis time.
- [ ] **White-Label Customization**: Custom domain mapping, brand logos, and customizable client portal styling.
