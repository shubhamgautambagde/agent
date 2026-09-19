# AI Operating Rules & Engineering Boundaries (Rules.md)

This document establishes the mandatory boundaries, constraints, and coding standards for any AI assistant or engineer contributing to the **Rapida Voice AI** repository.

---

## 1. Core Engineering Principles

- **KISS (Keep It Simple, Stupid)**: Choose the smallest complete solution. Avoid unnecessary moving parts or convoluted abstractions.
- **YAGNI (You Aren't Gonna Need It)**: Reject speculative interfaces, unused configuration flags, and anticipatory generalizations. Build only for verified current needs.
- **Strict Ownership**: Every file, state transition, goroutine, database connection, and test must have one explicit, unambiguous owner.
- **Single Source of Truth**: Never duplicate authoritative state, policies, credentials, or schema definitions.
- **Explicit Contracts**: Always define clear timeouts, cancellation mechanisms, and error signatures on APIs and gRPC interfaces.
- **Evidence Over Confidence**: Code is only complete when verified by passing tests, linters, and type checks.

---

## 2. Hard Boundaries & Forbidden Patterns

### 2.1 Vocabulary & Naming Prohibitions
- ❌ **No `normaliz*` Stem**: Do NOT introduce identifiers, functions, variables, comments, or documentation containing the stem `normaliz` (e.g., `normalize`, `normalizer`, `normalization`). Instead, use precise domain terms:
  - `parse`, `validate`, `trim`, `map`, `canonicalize`, `sanitize`, or `convert`.
- ❌ **No Generic Package Names**: NEVER create or add to packages named `util`, `utils`, `helper`, `helpers`, `manager`, `data`, or `process`. Keep logic close to the domain entity it operates upon.
- ❌ **No Redundant Prefixes**: Avoid `Get` prefixes on Go getters (e.g., use `session.Audio()` instead of `session.GetAudio()`).

### 2.2 Architectural Guardrails
- ❌ **No Direct HTTP Between Go Services**: All inter-service communication MUST use typed gRPC clients located in `pkg/clients/`. Never execute ad-hoc HTTP calls between internal services.
- ❌ **No Global Mutable State**: Hidden globals and implicit initialization (`init()` with side effects) are strictly disallowed. All dependencies must be injected at construction boundaries.
- ❌ **No Ignored Errors**: Swallowing errors (`_ = err`) or unhandled promise rejections is forbidden. Always handle, wrap, or explicitly log errors with contextual metadata.
- ❌ **No Redux in Frontend**: State management in the React UI is strictly managed via lightweight Zustand stores (`use-*-page-store.ts`) or React context.

---

## 3. Language & Runtime Standards

### 3.1 Go (v1.25)
- **Formatting & Imports**: Enforce `gofmt` and `goimports` ordering:
  1. Standard library
  2. External dependencies
  3. Internal packages (`github.com/rapidaai/...`)
- **Entity Model**: Every relational entity must compose GORM base models:
  - `Audited` (Snowflake ID generated in `BeforeCreate`, `created_at`, `updated_at`)
  - `Mutable` (`status`, `created_by`, `updated_by`)
  - `Organizational` (`organization_id`, `project_id`)
- **Logging**: Use Zap SugaredLogger exclusively via `commons.NewApplicationLogger`. Always include structured key-value pairs (e.g., `logger.Infow("session started", "session_id", id)`).
- **Concurrency & Goroutines**: Every spawned goroutine must have an associated `context.Context` for cancellation and structured panic recovery.

### 3.2 TypeScript & React (v18)
- **Typing**: Zero `any` policy. All API payloads, component props, and event handlers must be explicitly typed.
- **Naming Conventions**:
  - Components and types: `PascalCase`
  - Functions, variables, files, and hooks: `camelCase` (or kebab-case for file names)
  - React hooks: MUST start with `use` (e.g., `useVoiceStream`)
- **Styling**: Strictly adhere to the theme contract defined in `Design.md`. Use Tailwind utility classes; do not write arbitrary inline hex styles.
- **Testing**: Every UI component change should include or update tests in `ui/src/**/__tests__/`.

### 3.3 Python (FastAPI & Celery in `document-api`)
- **Typing**: Full type annotations required for all FastAPI request/response models via Pydantic.
- **Worker Safety**: Celery tasks must be idempotent, define explicit time limits (`time_limit`, `soft_time_limit`), and handle file descriptor cleanup during document ingestion.

---

## 4. Change Classification & Lifecycle Tiers

Before writing code, classify your work into one of the three established tiers:

1. **Fast Tier**:
   - *Use for*: Documentation, lint fixes, formatting, test-only adjustments, isolated zero-risk bug fixes.
   - *Flow*: `understand -> implement -> targeted verification`.
2. **Standard Tier**:
   - *Use for*: Feature development and bug fixes isolated within a single service or component.
   - *Flow*: `understand -> concise plan -> implement -> verify -> review`.
3. **Governed Tier**:
   - *Use for*: Public API changes, protobuf changes (`.proto`), database migrations, cross-service contracts, or authentication updates.
   - *Flow*: `understand -> draft RFC in rfcs/ -> review & accept -> implement -> verify`.

---

## 5. Verification Checklist Before Completion

Before claiming any task or PR is complete, run the corresponding validation commands:

- [ ] **Go Services**: `go test ./api/<service>/...` or `go test ./...`
- [ ] **Go Linting**: `golangci-lint run`
- [ ] **UI Type Check**: `cd ui && yarn checkTs`
- [ ] **UI Tests**: `cd ui && yarn test`
- [ ] **UI Linting**: `cd ui && yarn lint`
- [ ] **Commit Messages**: Follow Conventional Commits (`feat: add cartesia tts transformer`, `fix: handle vad silence timeout`).
