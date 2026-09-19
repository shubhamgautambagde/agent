# Project Requirements Document (PRD)

## 1. Executive Summary

**Rapida** is an open-source, production-grade voice AI orchestration platform engineered for enterprise scalability, deploy-anywhere flexibility, and multi-client ownership. Built in Go with gRPC, WebRTC, SIP, and a modern React console, Rapida delivers sub-second bidirectional voice conversational agents with complete provider independence (Bring-Your-Own-Model / Provider for STT, LLM, TTS, and Telephony).

---

## 2. Problem Statement & Vision

### Problem
- **Vendor Lock-in**: Proprietary voice platforms restrict choice of LLM, TTS voices, transcription engines, and telephony providers.
- **Latency & Reliability Gaps**: High latency across multi-vendor pipelines ruins natural human conversational cadence.
- **Data Sovereignty & Enterprise Compliance**: Regulated industries and agency clients cannot transmit raw audio or customer transcripts through opaque third-party clouds.
- **Operational Complexity**: Integrating VAD (Voice Activity Detection), End-of-Speech detection, interruption handling, and tool execution requires custom low-level plumbing.

### Vision
To provide the definitive open-source voice infrastructure platform enabling enterprises and agencies to deploy, white-label, monitor, and scale sovereign real-time voice agents on their own terms.

---

## 3. Target User Personas

| Persona | Role | Primary Needs |
| :--- | :--- | :--- |
| **Enterprise AI Architect** | Systems Architect / Lead | Self-hosted deployment, strict data residency, audit logs, high concurrency, zero vendor lock-in. |
| **Voice Application Developer** | Full-Stack / Go / Python Engineer | Clear gRPC/REST APIs, robust tool invocation, pluggable STT/TTS/LLM transformers, local dev environment. |
| **Agency Solutions Builder** | Technical Consultant / Agency Lead | White-label client deployments, multi-tenant workspace isolation, fast client onboarding, customizable UI. |
| **Voice Operations Lead** | AI Ops / Product Support | Real-time call tracing, latency breakdowns, transcript review, failure alerting, audio playback. |

---

## 4. Key Functional Capabilities

### 4.1 Real-Time Voice Orchestration (`assistant-api`)
- **Bidirectional Streaming**: Sub-300ms audio turnaround time via gRPC bidirectional streams.
- **Interruption Handling**: Instant speech interruption and barge-in detection via calibrated Voice Activity Detection (VAD) and End-of-Speech (EOS) algorithms.
- **Transformer Architecture**: Decoupled transformer interfaces for:
  - **STT (Speech-to-Text)**: Deepgram, Whisper, Google Speech, AssemblyAI, custom models.
  - **TTS (Text-to-Speech)**: ElevenLabs, Cartesia, PlayHT, OpenAI Audio, Azure Speech.
  - **VAD / Turn Detection**: Silero VAD, WebRTC VAD, configurable energy/silence thresholds.
- **Channel Delivery**:
  - **WebRTC**: Direct ultra-low-latency in-browser voice streaming.
  - **SIP / Telephony**: Inbound and outbound phone calls via Twilio, Vonage, Telnyx, or standard SIP trunks.
  - **WebSocket**: Universal streaming gateway for custom client applications.

### 4.2 LLM & Tool Orchestration (`integration-api` & `endpoint-api`)
- **Multi-Provider LLM Gateway**: OpenAI, Anthropic Claude, Google Gemini, Groq, Mistral, and local vLLM/Ollama endpoints.
- **Tool Calling & External Endpoints**: Dynamic function calling with retry logic, rate limits, caching, and fallback policies.
- **Context Management**: Dynamic prompt templating, variable substitution, and conversation memory retention.

### 4.3 Knowledge & RAG Pipeline (`document-api`)
- **Document Ingestion**: Parsing of PDF, DOCX, Markdown, and text files.
- **Chunking & Vectorization**: Semantic chunking with configurable embedding models.
- **OpenSearch Hybrid Search**: Tenant-isolated vector search with BM25 keyword matching for context injection during calls.

### 4.4 Organization & Identity Governance (`web-api`)
- **Multi-Tenancy**: Organization and project hierarchical boundaries.
- **Secure Vaults**: Encrypted storage for client API keys, SIP credentials, and provider secrets.
- **Role-Based Access Control (RBAC)**: Fine-grained permissions across workspaces and agent configurations.

### 4.5 Developer & Operator Console (`ui`)
- **Visual Assistant Builder**: No-code/low-code configuration of voice personas, prompts, tools, and fallback voices.
- **Call Session Observability**: Live call monitoring, full audio playback, interactive transcript timelines, and latency waterfalls.
- **Evaluation & Test Bench**: Interactive browser playground for real-time microphone testing and prompt iteration.

---

## 5. Non-Functional Requirements

### 5.1 Performance & Latency
- **Time-to-First-Audio (TTFA)**: < 450ms end-to-end target (User silence -> STT -> LLM token 1 -> TTS chunk 1 -> audio playback).
- **Service Overhead**: Internal microservice routing overhead < 20ms using binary gRPC.
- **Concurrency**: Minimum 500 concurrent live voice sessions per cluster node under standard hardware profiles.

### 5.2 Reliability & Availability
- **Graceful Degradation**: Automated fallback to secondary TTS voice or LLM model upon upstream provider timeout or error.
- **Self-Healing WebSockets / gRPC**: Reconnection handshakes with session restoration.
- **Zero-Downtime Deployments**: Stateless service tier (`assistant-api`, `web-api`, `integration-api`) backed by persistent Redis and PostgreSQL.

### 5.3 Security & Compliance
- **Data Sovereignty**: Complete on-premise or sovereign cloud deployment capability.
- **Encryption**: TLS 1.3 in transit; AES-256 GCM encryption at rest for secrets in Vaults.
- **PII Scrubbing**: Optional real-time transcript redaction before storage.

---

## 6. Assumptions & Dependencies

- **Runtime Prerequisites**: Go 1.25+, Node.js 22+, Python 3.11+, Docker Compose / Kubernetes.
- **Databases**: PostgreSQL 15 (GORM), Redis 7 (caching/pubsub), OpenSearch 2.11 (vector index).
- **External Providers**: Valid API keys for upstream providers configured via the platform's Vault or environment files.
