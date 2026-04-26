# Renai Game LLM MVP - Implementation Blueprint

## Document Control
- Status: Draft Implementation Blueprint
- Version: v1.2.2
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.2.2 | Minor | Expanded the planning baseline into a strict scaffolding blueprint with exact React and Go file layout, naming conventions, and initial environment or secret keys for PostgreSQL, Redis, RabbitMQ, Facebook auth, and the LLM provider. | Backlog M0 and M1 work can start directly from concrete file targets and env-template contracts instead of deriving them ad hoc during implementation. |
| 2026-04-26 | v1.2.1 | Minor | Locked the framework and session implementation choices inside the selected stack baseline by choosing a Vite-based React toolchain, chi-based Go services, Redis-backed opaque cookie sessions, and managed-secret injection for backend runtimes. | Backlog M0 tasks, contract assumptions, and the task artifact should treat framework and session choices as resolved and encode them directly into scaffolding and config work. |
| 2026-04-26 | v1.2.0 | Major | Locked the phase 1 technology baseline to React for the browser client, Go for backend API and worker services, PostgreSQL for relational storage, Redis for cache, and RabbitMQ for queueing. | API contracts, execution backlog, task artifact, and implementation kickoff work must use the selected stack baseline instead of treating stack choice as open. |
| 2026-04-26 | v1.1.0 | Major | Reconciled the implementation blueprint to PRD v3.0.2 and architecture v1.1.0 by removing guest mode, adopting Player and MC terminology, enforcing authenticated-only chat, and encoding the approved content, retention, and deletion posture. | API contracts, backlog sequencing, and the machine-readable task artifact must align to the authenticated-only phase 1 baseline and internal lifecycle workflow. |
| 2026-04-26 | v1.0.0 | Major | Established the implementation blueprint as the managed planning baseline for the MVP. | API contracts, backlog updates, and implementor execution should continue to align to Planning v1.0.0 until a later planning revision is published. |

## Upstream Baseline
- Based On: MVP PRD v3.0.2 and phase 1 architecture packet v1.1.0

## Executive Summary
This document translates the approved MVP product and architecture into an implementation blueprint with a selected phase 1 technology baseline. The design target remains a responsive browser client plus a modular monolith backend and an async worker, deployed on a medium-sized phase 1 environment.

The current planning baseline is materially simpler than the earlier draft:
- chat is authenticated-only
- `Player` is the logged-in user identity
- `MC` is the AI character identity
- frontend browser client uses React
- backend API and worker services use Go
- relational persistence uses PostgreSQL
- cache uses Redis
- queueing uses RabbitMQ
- hosted-model behavior is non-explicit in phase 1
- retention windows are fixed
- delete is internal or admin-managed rather than Player self-service

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/renai-game-llm-mvp-architecture-overview.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-erd.md`
- `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md`
- `docs/02_architecture/renai-game-llm-mvp-component-diagrams.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`
- `docs/02_architecture/renai-game-llm-mvp-security-architecture.md`

## Scope
This blueprint covers:
- phase 1 browser client
- backend API / BFF
- Facebook login and authenticated session handling
- authenticated Player-to-MC chat orchestration and delayed replies
- relationship-state engine
- short-term and long-term memory pipeline
- policy and safety enforcement for hosted non-explicit operation
- internal or admin-managed delete workflow
- phase 1 deployment skeleton
- tests and rollout sequencing

This blueprint does not cover:
- guest or anonymous chat sessions
- Player-facing delete controls
- explicit adult sexual content in phase 1
- native mobile apps
- premium score visibility in phase 1
- shared-world memory or cross-MC memory

## Applied Planning Artifacts
This blueprint is complemented by these execution-oriented artifacts:
- `docs/03_implementation/renai-game-llm-mvp-api-contract-drafts.md`
- `docs/03_implementation/renai-game-llm-mvp-execution-backlog.md`
- `docs/03_implementation/renai-game-llm-mvp-task-artifact.json`

## Selected Phase 1 Technology Baseline
| Layer | Selected Technology |
| --- | --- |
| Browser client | React |
| Backend API / BFF | Go |
| Async worker | Go |
| Relational database | PostgreSQL |
| Cache | Redis |
| Queue | RabbitMQ |

### Mapping Notes
- React is the browser-client baseline for discovery, authentication, chat, and restore views.
- Go is the service baseline for both the browser-facing API and the async worker runtime.
- PostgreSQL remains the source of truth for Players, MCs, conversations, messages, memory, relationship state, reply schedules, and audit traces.
- Redis is the cache layer for transient coordination, rate-limit state, and other low-latency ephemeral lookups.
- RabbitMQ is the queueing layer for delayed replies, background memory updates, audit jobs, and purge jobs.

## Selected Framework And Library Baseline
| Concern | Selected Tooling |
| --- | --- |
| React browser toolchain | React + TypeScript + Vite |
| Client routing | React Router |
| Client server-state fetching | TanStack Query |
| Client local UI state | Zustand |
| Client unit/component tests | Vitest + Testing Library |
| Client end-to-end tests | Playwright |
| Go HTTP stack | `net/http` + `chi` |
| Request validation | `go-playground/validator` |
| PostgreSQL access | `pgx` + `sqlc` |
| Schema migrations | `goose` |
| Redis client | `go-redis` |
| RabbitMQ client | `amqp091-go` |
| Logging | `log/slog` |
| Go test assertions | `testify` |

### Selection Notes
- Vite is preferred because the current MVP does not require server-rendered React to satisfy the approved product scope.
- React Router, TanStack Query, and Zustand fit the current split between route-driven screens, server-owned chat state, and small local UI stores.
- `chi` keeps the Go HTTP layer close to the standard library and avoids a heavier framework surface for the MVP.
- `pgx` plus `sqlc` keeps PostgreSQL access explicit and typed, which fits the conversation-centric schema and worker-heavy background jobs better than an ORM-first approach.
- `goose`, `go-redis`, and `amqp091-go` match the selected infrastructure baseline without adding unnecessary abstraction.

## Selected Session And Secret-Distribution Approach
### Session Transport
- Use a secure, `HttpOnly`, `SameSite=Lax` cookie carrying an opaque server-generated session ID.
- Do not store auth tokens in `localStorage` or expose provider tokens to the React client.
- Keep session state server-side in Redis so runtime invalidation and session-adjacent checks remain consistent across the API and worker layers.
- Enforce `Origin` validation on state-changing requests in addition to secure cookie settings.

### Secret Distribution
- Store production and staging secrets in a managed secret store outside the repository.
- Inject secrets into the Go backend API and Go worker as environment variables at deploy or container start time.
- Keep frontend runtime config limited to non-secret public values only.
- Use non-committed local `.env` files only for development and local docker-compose workflows.

## Implementation Approach
### Recommended Delivery Shape
- React `web-client` application for the responsive browser UI
- Go `backend-api` service for browser-facing API and chat orchestration
- Go `worker` service for scheduled reply execution and background updates
- PostgreSQL relational datastore
- Redis cache layer
- RabbitMQ queue layer
- containerized phase 1 deployment

### Guiding Rules
- Require an authenticated Player session before any chat creation, restore, or message send operation.
- Preserve one durable conversation root per Player-MC pair.
- Make delayed reply scheduling a backend-owned concern.
- Keep provider-specific logic isolated behind a single adapter layer.
- Keep all memory and relationship state strictly scoped to the owning conversation.
- Enforce hosted-model non-explicit behavior in the policy layer, not in the client.
- Treat lifecycle enforcement and purge execution as security and privacy obligations, not optional cleanup.
- Keep provider, infrastructure, and privileged-access secrets server-side only.

## Proposed Top-Level File Tree
The paths below are proposed and currently do not exist.

```text
apps/
  web-client/
    app-shell/
    routes/
      home/
      mc-discovery/
      chat/
      auth/
    components/
      auth-required-gate/
      delayed-message-status/
      conversation-list/
      mc-profile-card/
    services/
      api-client/
      session-client/
      auth-client/
      chat-client/
    state/
      session-store/
      conversation-store/
      mc-store/

services/
  backend-api/
    app-entry/
    config/
    modules/
      identity/
      mc-catalog/
      conversations/
      messages/
      chat-orchestration/
      memory/
      relationship/
      policy-safety/
      provider/
      audit/
      admin-lifecycle/
    persistence/
      repositories/
      migrations/
      models/
    contracts/
      http/
      jobs/
    tests/

  worker/
    worker-entry/
    jobs/
      execute-delayed-reply/
      update-short-term-context/
      update-long-term-memory/
      update-relationship-state/
      purge-conversation/
      emit-audit-events/
    tests/

infra/
  phase1/
    env-template/
    docker-compose/
    app-config/
    queue-config/
    db-config/

tests/
  end-to-end/
  integration/
  fixtures/
```

## File-By-File Responsibilities
### Client Layer
| Proposed Path | Responsibility |
| --- | --- |
| `apps/web-client/app-shell/` | Browser bootstrap, route mounting, global layout, and session hydration |
| `apps/web-client/routes/home/` | Landing screen and login-forward entry points |
| `apps/web-client/routes/mc-discovery/` | Mobile swipe and desktop list discovery views for MC selection |
| `apps/web-client/routes/chat/` | One-on-one Player-to-MC chat screen and delayed-message state |
| `apps/web-client/routes/auth/` | Facebook login initiation and callback handling |
| `apps/web-client/components/auth-required-gate/` | Client-side login-required UX for chat entry |
| `apps/web-client/components/delayed-message-status/` | Simple delayed-reply visual state |
| `apps/web-client/components/conversation-list/` | Returning Player conversation list and resume UX |
| `apps/web-client/components/mc-profile-card/` | Reusable MC profile rendering for discovery surfaces |
| `apps/web-client/services/api-client/` | Shared HTTP request layer |
| `apps/web-client/services/session-client/` | Session inspection and auth-state refresh |
| `apps/web-client/services/auth-client/` | Facebook auth exchange support |
| `apps/web-client/services/chat-client/` | Conversation load, message send, and refresh behavior |
| `apps/web-client/state/session-store/` | Authenticated session state and login-required flags |
| `apps/web-client/state/conversation-store/` | Active conversation UI state |
| `apps/web-client/state/mc-store/` | MC discovery and profile state |

### Backend API Layer
| Proposed Path | Responsibility |
| --- | --- |
| `services/backend-api/app-entry/` | Service bootstrap and module registration |
| `services/backend-api/config/` | Runtime configuration loading including lifecycle constants |
| `services/backend-api/modules/identity/` | Facebook auth resolution, internal Player identity mapping, and session handling |
| `services/backend-api/modules/mc-catalog/` | Aira, Lyra, and Nova profile retrieval and future catalog support |
| `services/backend-api/modules/conversations/` | Conversation lookup, creation, reopen, lifecycle checks, and ownership rules |
| `services/backend-api/modules/messages/` | Message persistence and retrieval |
| `services/backend-api/modules/chat-orchestration/` | Main request-path coordination across policy, memory, relationship, and scheduling |
| `services/backend-api/modules/memory/` | Context assembly and memory update orchestration |
| `services/backend-api/modules/relationship/` | Score loading, updates, and threshold evaluation |
| `services/backend-api/modules/policy-safety/` | Non-explicit hosted-model policy, hard guardrails, and post-generation validation |
| `services/backend-api/modules/provider/` | Hosted provider adapter and request normalization |
| `services/backend-api/modules/audit/` | Metadata-first audit event recording and review hooks |
| `services/backend-api/modules/admin-lifecycle/` | Internal or admin-managed delete intake and purge dispatch |
| `services/backend-api/persistence/repositories/` | Data access boundaries per entity |
| `services/backend-api/persistence/migrations/` | Schema migration definitions |
| `services/backend-api/persistence/models/` | Persistence entity definitions aligned to the ERD |
| `services/backend-api/contracts/http/` | Request and response contract types or schemas |
| `services/backend-api/contracts/jobs/` | Worker job payload schemas |
| `services/backend-api/tests/` | Module-level and API integration tests |

### Worker Layer
| Proposed Path | Responsibility |
| --- | --- |
| `services/worker/worker-entry/` | Worker bootstrap and queue bindings |
| `services/worker/jobs/execute-delayed-reply/` | Execute scheduled reply, call provider, and persist MC message |
| `services/worker/jobs/update-short-term-context/` | Refresh rolling recent context |
| `services/worker/jobs/update-long-term-memory/` | Create or compact long-term memory entries |
| `services/worker/jobs/update-relationship-state/` | Apply relationship score deltas and thresholds |
| `services/worker/jobs/purge-conversation/` | Hard-delete owner-bound transcript and derived state |
| `services/worker/jobs/emit-audit-events/` | Persist background audit and trace events |
| `services/worker/tests/` | Worker flow, retry, scheduling, and purge tests |

### Infra Layer
| Proposed Path | Responsibility |
| --- | --- |
| `infra/phase1/env-template/` | Environment variable template including retention constants |
| `infra/phase1/docker-compose/` | Phase 1 multi-container local or single-host orchestration |
| `infra/phase1/app-config/` | App runtime configuration examples |
| `infra/phase1/queue-config/` | Queue/cache setup defaults |
| `infra/phase1/db-config/` | Database initialization defaults |

## Exact Scaffolding File Plan
### Browser Client Exact File Layout
```text
apps/web-client/
  package.json
  tsconfig.json
  vite.config.ts
  vitest.config.ts
  playwright.config.ts
  index.html
  .env.example
  src/
    main.tsx
    app/
      App.tsx
      router.tsx
      providers.tsx
      queryClient.ts
    routes/
      home/HomePage.tsx
      mc-discovery/McDiscoveryPage.tsx
      chat/ChatPage.tsx
      auth/AuthCallbackPage.tsx
    components/
      auth/AuthRequiredGate.tsx
      chat/DelayedMessageStatus.tsx
      chat/ConversationList.tsx
      mc/McProfileCard.tsx
    services/
      http/apiClient.ts
      auth/sessionApi.ts
      auth/facebookAuth.ts
      chat/chatApi.ts
    state/
      sessionStore.ts
      conversationStore.ts
      mcStore.ts
    styles/
      tokens.css
    test/
      setup.ts
```

Key responsibilities:
- `package.json` owns the React, Vite, routing, query, state, and test dependency baseline.
- `src/app/router.tsx` is the only route registration root.
- `src/services/http/apiClient.ts` is the only place that sets `credentials: include` and shared HTTP defaults.
- `src/state/*Store.ts` files are limited to UI-local state; server state stays in TanStack Query.

### Backend API Exact File Layout
```text
services/backend-api/
  go.mod
  go.sum
  .env.example
  cmd/
    api/main.go
  internal/
    app/app.go
    config/config.go
    platform/
      logging/logger.go
      postgres/pool.go
      redis/client.go
      rabbitmq/publisher.go
      session/store.go
      facebook/client.go
      llm/client.go
    http/
      router/router.go
      middleware/
        origin/origin.go
        session/session.go
        requestid/request_id.go
      handlers/
        session/handler.go
        auth/handler.go
        mc/handler.go
        conversation/handler.go
        message/handler.go
        internal_lifecycle/handler.go
    modules/
      identity/service.go
      mc_catalog/service.go
      conversations/service.go
      messages/service.go
      chat_orchestration/service.go
      memory/service.go
      relationship/service.go
      policy_safety/service.go
      provider/service.go
      audit/service.go
      admin_lifecycle/service.go
  db/
    migrations/
      0001_initial_schema.sql
    queries/
      conversations.sql
      messages.sql
      memory.sql
      relationship.sql
      audit.sql
  sqlc.yaml
  tests/
    integration/
```

Key responsibilities:
- `cmd/api/main.go` only boots config, infrastructure clients, HTTP router, and graceful shutdown.
- `internal/platform/*` owns infrastructure adapters and contains no business rules.
- `internal/modules/*/service.go` owns business rules and orchestration per module.
- `db/migrations/` and `db/queries/` are the single source of truth for PostgreSQL schema and query generation.

### Worker Exact File Layout
```text
services/worker/
  go.mod
  go.sum
  .env.example
  cmd/
    worker/main.go
  internal/
    app/worker.go
    config/config.go
    platform/
      logging/logger.go
      postgres/pool.go
      redis/client.go
      rabbitmq/consumer.go
      llm/client.go
    jobs/
      execute_delayed_reply/handler.go
      update_short_term_context/handler.go
      update_long_term_memory/handler.go
      update_relationship_state/handler.go
      purge_conversation/handler.go
      emit_audit_events/handler.go
  tests/
    integration/
```

Key responsibilities:
- `cmd/worker/main.go` only wires config, infrastructure, queue subscriptions, and shutdown.
- `internal/jobs/*/handler.go` owns one queue job consumer per job family.
- Worker infrastructure packages mirror API naming so operational ownership stays predictable across services.

### Infrastructure Exact File Layout
```text
infra/phase1/
  env-template/
    web-client.env.example
    backend-api.env.example
    worker.env.example
    shared.infra.env.example
  docker-compose/
    docker-compose.yml
  app-config/
    backend-api.config.example.yaml
    worker.config.example.yaml
  queue-config/
    rabbitmq.definitions.example.json
  db-config/
    postgres.init.example.sql
```

## Naming And Layout Conventions
### React Naming Rules
- Route folders use kebab-case such as `mc-discovery`.
- React component files use PascalCase such as `ChatPage.tsx` and `AuthRequiredGate.tsx`.
- Store files use camelCase ending with `Store.ts`.
- API wrapper files use camelCase ending with `Api.ts` or a clear auth-specific name such as `facebookAuth.ts`.
- Shared design tokens live in `src/styles/tokens.css`; component-scoped styles should use `ComponentName.module.css` only when plain composition is insufficient.

### Go Naming Rules
- Keep one `go.mod` per deployable service in phase 1: one under `services/backend-api/` and one under `services/worker/`.
- Entry points always live under `cmd/<binary>/main.go`.
- Business logic lives under `internal/modules/<module>/service.go`.
- HTTP handlers live under `internal/http/handlers/<resource>/handler.go`.
- Package names stay lowercase and avoid hyphens.
- SQL migrations use zero-padded numeric prefixes such as `0001_initial_schema.sql`.
- Queue job directories mirror queue names using snake_case such as `execute_delayed_reply`.

### Infra Naming Rules
- Env templates end with `.env.example` and never contain real secrets.
- Example runtime config files end with `.config.example.yaml`.
- Queue and database bootstrap artifacts use `.example.*` suffixes when they are not directly executable production files.

## Initial Environment And Secret Baseline
### Public Browser Environment Keys
| Key | Secret | Purpose |
| --- | --- | --- |
| `VITE_APP_ENV` | No | Browser runtime environment name such as `local`, `staging`, or `production` |
| `VITE_API_BASE_URL` | No | Browser-visible API base URL |
| `VITE_FACEBOOK_APP_ID` | No | Facebook application identifier used by the browser auth flow |

### Backend API Environment Keys
| Key | Secret | Purpose |
| --- | --- | --- |
| `APP_ENV` | No | Runtime environment name |
| `HTTP_PORT` | No | Backend API listen port |
| `PUBLIC_APP_ORIGIN` | No | Allowed browser origin for cookie and origin validation |
| `SESSION_COOKIE_NAME` | No | Cookie name for the opaque session identifier |
| `SESSION_COOKIE_DOMAIN` | No | Cookie domain scope |
| `SESSION_COOKIE_SECURE` | No | Enables `Secure` cookies outside local development |
| `SESSION_TTL_HOURS` | No | Session lifetime policy |
| `SESSION_REDIS_PREFIX` | No | Redis key prefix for session records |
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `REDIS_URL` | Yes | Redis connection string |
| `RABBITMQ_URL` | Yes | RabbitMQ connection string |
| `FACEBOOK_APP_ID` | No | Server-side Facebook app identifier for token verification |
| `FACEBOOK_APP_SECRET` | Yes | Server-side Facebook app secret |
| `LLM_PROVIDER` | No | Hosted provider identifier such as `gemini` |
| `LLM_MODEL` | No | Hosted model name used in phase 1 |
| `LLM_API_KEY` | Yes | Hosted provider API key |
| `CONVERSATION_RETENTION_DAYS` | No | Expected default `365` |
| `DERIVED_DELETE_WINDOW_DAYS` | No | Expected default `30` |
| `AUDIT_RETENTION_DAYS` | No | Expected default `90` |
| `INTERNAL_ADMIN_SHARED_SECRET` | Yes | Shared secret for privileged internal lifecycle endpoints until a stronger admin identity layer exists |

### Worker Environment Keys
| Key | Secret | Purpose |
| --- | --- | --- |
| `APP_ENV` | No | Runtime environment name |
| `WORKER_CONCURRENCY` | No | Worker concurrency limit |
| `DATABASE_URL` | Yes | PostgreSQL connection string |
| `REDIS_URL` | Yes | Redis connection string |
| `RABBITMQ_URL` | Yes | RabbitMQ connection string |
| `LLM_PROVIDER` | No | Hosted provider identifier |
| `LLM_MODEL` | No | Hosted model name |
| `LLM_API_KEY` | Yes | Hosted provider API key |
| `CONVERSATION_RETENTION_DAYS` | No | Expected default `365` |
| `DERIVED_DELETE_WINDOW_DAYS` | No | Expected default `30` |
| `AUDIT_RETENTION_DAYS` | No | Expected default `90` |

### Shared Infrastructure Bootstrap Keys
| Key | Secret | Purpose |
| --- | --- | --- |
| `POSTGRES_DB` | No | Default local database name for docker-compose bootstrap |
| `POSTGRES_USER` | No | Default local postgres username |
| `POSTGRES_PASSWORD` | Yes | Default local postgres password |
| `REDIS_PASSWORD` | Yes | Optional local redis password when enabled |
| `RABBITMQ_DEFAULT_USER` | No | Local rabbitmq username |
| `RABBITMQ_DEFAULT_PASS` | Yes | Local rabbitmq password |
| `RABBITMQ_DEFAULT_VHOST` | No | Local rabbitmq vhost |

## Module Contracts
### Identity Module
Responsibilities:
- resolve Facebook-authenticated Players
- store and refresh session state
- expose authenticated `player_account_id` to downstream modules
- deny chat operations when authentication is absent or expired

Inputs:
- browser session inspection request
- Facebook auth callback payload

Outputs:
- anonymous-or-authenticated session view
- authenticated session bound to `player_account_id`

### Conversations Module
Responsibilities:
- resolve the active conversation for one Player-MC pair
- enforce ownership and lifecycle state on read and write operations
- support returning Player restore behavior

### Chat Orchestration Module
Responsibilities:
- accept Player messages
- load MC profile, memory, relationship, and policy context
- persist Player messages
- schedule delayed reply jobs
- coordinate post-reply background jobs

### Memory Module
Responsibilities:
- load prompt-ready short-term and long-term context
- maintain conversation-scoped memory only
- asynchronously write updated summaries and salient facts

### Relationship Module
Responsibilities:
- load and update deterministic relationship metrics
- compute tone and delay hints
- ensure score bounds remain valid

### Policy And Safety Module
Responsibilities:
- apply hard guardrails
- enforce hosted-model non-explicit phase 1 behavior
- validate model output before persistence
- emit policy-related audit traces when blocking or constraining output

### Provider Module
Responsibilities:
- translate internal prompt layers into provider requests
- normalize provider responses
- isolate hosted provider SDK or API specifics

### Audit And Admin Lifecycle Modules
Responsibilities:
- record metadata-first operational and safety traces
- accept internal or admin-managed delete requests
- enqueue purge work
- avoid exposing transcript-heavy privileged access by default

## Data Changes Required
The implementation must realize at least these persistent entities from the ERD:
- `PlayerAccount`
- `IdentityLink`
- `MCProfile`
- `Conversation`
- `Message`
- `ShortTermContext`
- `LongTermMemory`
- `RelationshipState`
- `ReplySchedule`
- `AuditTrace`

### Required Core Constraints
- one active conversation per Player-MC pair
- score values bounded between 0 and 100
- no cross-MC memory references
- derived state owned by the conversation root

### Required Lifecycle And Security Rules
- authentication is required before chat creation, restore, or send
- non-active conversations must reject new writes and delayed replies
- derived memory and relationship state must cascade from conversation lifecycle
- provider credentials must remain unavailable to the browser client
- audit payloads should remain metadata-first and minimized
- delete is internal or admin-managed only in phase 1

## Dependency Order
### Wave 0: Decision Lock And Runtime Baseline
- encode the selected React + TypeScript + Vite client baseline and the selected Go service libraries into repo conventions
- encode the selected cookie-session and managed-secret distribution approach into runtime config conventions
- encode approved lifecycle windows and admin-delete operating model
- create repo scaffolding and environment template baseline

### Wave 1: Foundation
- Facebook login support
- MC catalog
- conversation and message persistence
- browser shell and discovery screens

### Wave 2: Core Chat Loop
- session inspection and auth exchange
- authenticated chat request handling
- delayed reply scheduling
- provider adapter
- basic message retrieval and chat rendering

### Wave 3: Memory And Relationship State
- short-term context storage
- long-term memory summarization jobs
- relationship score engine and thresholds
- continuity and restore flow for one Player-MC pair

### Wave 4: Policy And Hardening
- hosted-model non-explicit policy enforcement
- admin-managed lifecycle delete flow
- audit events
- observability, rate controls, and retry handling

## API And Job Surface Expectations
The implementation should align with a companion contract draft that includes:
- session inspection
- Facebook auth exchange
- MC listing and detail retrieval
- conversation resolve and restore
- message send and refresh
- delayed reply execution jobs
- memory-update jobs
- relationship-update jobs
- internal lifecycle delete request and purge jobs

## Testing Expectations
### Required Test Areas
- authenticated session enforcement before chat access
- Facebook-authenticated conversation restore
- one-Player-one-MC memory isolation
- delayed reply scheduling and delivery
- relationship score bounds and threshold behavior
- hard-guardrail refusal behavior
- hosted-model non-explicit policy behavior
- lifecycle blocking for deleted or purge-pending conversations
- internal delete to purge completion flow

### Recommended Test Layers
- module unit tests
- API integration tests
- worker job tests
- end-to-end browser tests for login, discovery, chat, and restore flows

## Rollout Order
1. Internal seeded environment with Aira, Lyra, and Nova
2. Authenticated Player login and chat smoke test
3. Delayed reply and memory validation pass
4. Safety, lifecycle, and policy review pass
5. Cost and latency observation pass

## Open Questions
No open questions are currently blocking the planning baseline.

## Recommendation Summary
Recommendation:
Implement the MVP as a modular monolith plus async worker using React + TypeScript + Vite for the browser client, `chi`-based Go services with `pgx` and `sqlc`, PostgreSQL for durable data, Redis for cache and session-state concerns, and RabbitMQ for job delivery. Keep chat authenticated-only, keep Player-MC continuity rooted in one durable conversation, keep provider integration isolated, and treat lifecycle enforcement plus internal delete handling as part of the first release baseline rather than a later follow-up.
