# Renai Game LLM MVP - Implementation Blueprint

## Document Control
- Status: Draft Implementation Blueprint
- Version: v1.0
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.0 | Major | Established the implementation blueprint as the managed planning baseline for the MVP. | API contracts, backlog updates, and implementor execution should continue to align to Planning v1.0 until a later planning revision is published. |

## Upstream Baseline
- Based On: MVP PRD v1.0 and phase 1 architecture packet v1.0

## Executive Summary
This document translates the approved MVP product and architecture into a strict implementation blueprint for engineering handoff. The design target is a responsive browser client plus a modular monolith backend and an async worker, deployed on a medium-sized phase 1 environment.

The blueprint is intentionally stack-neutral because the repository does not yet define a concrete application stack. All proposed files and directories below are logical implementation targets and should be mapped into the team’s chosen framework without changing the architecture or feature boundaries.

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
- guest trial and Facebook login flow
- chat orchestration and delayed replies
- relationship state engine
- short-term and long-term memory pipeline
- policy and safety enforcement
- phase 1 deployment skeleton
- tests and rollout sequencing

This blueprint does not cover:
- native mobile apps
- premium score visibility in phase 1
- multi-provider linking beyond the future-ready identity shape
- shared-world memory or cross-character memory
- production-grade explicit adult-content enablement on hosted models

## Applied Planning Artifacts
This blueprint is complemented by these execution-oriented artifacts:
- `docs/03_implementation/renai-game-llm-mvp-execution-backlog.md`
- `docs/03_implementation/renai-game-llm-mvp-task-artifact.json`

## Implementation Approach
### Recommended Delivery Shape
- `web-client` application for the responsive browser UI
- `backend-api` service for browser-facing API and chat orchestration
- `worker` service for scheduled reply execution and background updates
- relational datastore
- queue/cache layer
- containerized phase 1 deployment

### Guiding Rules
- Keep the codebase modular but deployable as a simple phase 1 system.
- Preserve strict conversation-scoped memory boundaries.
- Make delayed reply scheduling a backend-owned concern.
- Keep provider-specific logic isolated behind a single adapter layer.
- Treat guest chat state as disposable and non-migrating.
- Enforce conversation lifecycle state before reply delivery, restore, or background update.
- Keep provider, infrastructure, and privileged-access secrets server-side only.

## Proposed Top-Level File Tree
The paths below are proposed and currently do not exist.

```text
apps/
  web-client/
    app-shell/
    routes/
      home/
      character-discovery/
      chat/
      auth/
    components/
      age-confirmation-dialog/
      login-gate/
      guest-limit-modal/
      delayed-message-status/
    services/
      api-client/
      session-client/
      chat-client/
    state/
      session-store/
      conversation-store/
      character-store/
      policy-store/

services/
  backend-api/
    app-entry/
    config/
    modules/
      identity/
      guest-trial/
      character-catalog/
      conversations/
      messages/
      chat-orchestration/
      memory/
      relationship/
      policy-safety/
      provider/
      audit/
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
| `apps/web-client/routes/home/` | Landing screen, guest entry, and login CTA |
| `apps/web-client/routes/character-discovery/` | Mobile swipe and desktop list discovery views |
| `apps/web-client/routes/chat/` | One-on-one chat screen, delayed-message state, and login interruption handling |
| `apps/web-client/routes/auth/` | Facebook login initiation and callback handling |
| `apps/web-client/components/age-confirmation-dialog/` | Self-attested 18+ confirmation UI |
| `apps/web-client/components/login-gate/` | Guest-limit reached prompt and login requirement UX |
| `apps/web-client/components/guest-limit-modal/` | Explicit messaging for the 10-input guest cap |
| `apps/web-client/components/delayed-message-status/` | Simple delayed-reply visual state |
| `apps/web-client/services/api-client/` | Shared HTTP request layer |
| `apps/web-client/services/session-client/` | Session bootstrap, guest session creation, and auth-state refresh |
| `apps/web-client/services/chat-client/` | Message send, conversation fetch, and polling or refresh behavior |
| `apps/web-client/state/session-store/` | Guest/auth state and age-confirmation state |
| `apps/web-client/state/conversation-store/` | Active conversation UI state |
| `apps/web-client/state/character-store/` | Character discovery and profile state |
| `apps/web-client/state/policy-store/` | Local view of gating and policy flags |

### Backend API Layer
| Proposed Path | Responsibility |
| --- | --- |
| `services/backend-api/app-entry/` | Service bootstrap and module registration |
| `services/backend-api/config/` | Runtime configuration loading |
| `services/backend-api/modules/identity/` | Guest creation, Facebook auth resolution, internal account mapping |
| `services/backend-api/modules/guest-trial/` | 10-input cap enforcement and guest reset policy |
| `services/backend-api/modules/character-catalog/` | Aira, Lyra, Nova profile retrieval and future catalog support |
| `services/backend-api/modules/conversations/` | Conversation lookup, creation, reopen, and ownership rules |
| `services/backend-api/modules/messages/` | Message persistence and retrieval |
| `services/backend-api/modules/chat-orchestration/` | Main request-path coordination across policy, memory, relationship, and scheduling |
| `services/backend-api/modules/memory/` | Context assembly and memory update orchestration |
| `services/backend-api/modules/relationship/` | Score loading, updates, and threshold evaluation |
| `services/backend-api/modules/policy-safety/` | Age gate checks, hard guardrails, and hosted-safe-mode enforcement |
| `services/backend-api/modules/provider/` | Hosted provider adapter and request normalization |
| `services/backend-api/modules/audit/` | Audit event recording and query hooks |
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
| `services/worker/jobs/execute-delayed-reply/` | Execute scheduled reply, call provider, and persist character message |
| `services/worker/jobs/update-short-term-context/` | Refresh rolling recent context |
| `services/worker/jobs/update-long-term-memory/` | Create or compact long-term memory entries |
| `services/worker/jobs/update-relationship-state/` | Apply relationship score deltas and thresholds |
| `services/worker/jobs/emit-audit-events/` | Persist background audit and trace events |
| `services/worker/tests/` | Worker flow, retry, and scheduling tests |

### Infra Layer
| Proposed Path | Responsibility |
| --- | --- |
| `infra/phase1/env-template/` | Environment variable template |
| `infra/phase1/docker-compose/` | Phase 1 multi-container local or single-host orchestration |
| `infra/phase1/app-config/` | App runtime configuration examples |
| `infra/phase1/queue-config/` | Queue/cache setup defaults |
| `infra/phase1/db-config/` | Database initialization defaults |

## Module Contracts
### Identity Module
Responsibilities:
- create guest sessions
- resolve Facebook-authenticated users
- store self-attested 18+ confirmation timestamp
- expose authenticated user identity to downstream modules

Inputs:
- browser bootstrap request
- Facebook auth callback payload
- age-confirmation action

Outputs:
- guest session
- authenticated session
- internal `user_account_id` or `guest_session_id`

### Guest Trial Module
Responsibilities:
- count guest user input messages
- enforce 10-input cap
- block further guest sends when the cap is reached
- ensure guest state is not migrated at login

### Chat Orchestration Module
Responsibilities:
- accept user messages
- load character, memory, relationship, and policy context
- persist user message
- schedule delayed reply job

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
- enforce self-attested 18+ gate
- support hosted-safe-mode fallback
- validate model output before persistence

### Provider Module
Responsibilities:
- translate internal prompt layers into provider requests
- normalize provider responses
- isolate hosted provider SDK or API specifics

## Data Changes Required
The implementation must realize at least these persistent entities from the ERD:
- `UserAccount`
- `IdentityLink`
- `GuestSession`
- `CharacterProfile`
- `Conversation`
- `Message`
- `ShortTermContext`
- `LongTermMemory`
- `RelationshipState`
- `ReplySchedule`
- `AuditEvent`

### Required Core Constraints
- exactly one owner on each conversation: guest or authenticated user
- score values bounded between 0 and 100
- guest input count bounded to non-negative integers
- no cross-character memory references

### Required Lifecycle And Security Rules
- guest session expiry must be enforced by API and worker paths
- non-active conversations must reject new writes and delayed replies
- derived memory and relationship state must cascade from conversation lifecycle
- provider credentials must remain unavailable to the browser client
- audit payloads should remain metadata-first and minimized

## Dependency Order
### Wave 0: Decision Lock And Runtime Baseline
- confirm implementation stack mapping for web client, backend API, worker, relational store, and queue or cache layer
- confirm session mechanism and secret-distribution approach
- confirm retention-duration and delete-exposure policy values
- create repo scaffolding and environment template baseline

### Wave 1: Foundation
- guest session support
- Facebook login support
- character catalog
- conversation and message persistence
- browser shell and discovery screens

### Wave 2: Core Chat Loop
- chat request handling
- delayed reply scheduling
- provider adapter
- basic message retrieval and chat rendering

### Wave 3: Memory And Relationship State
- short-term context storage
- long-term memory summarization jobs
- relationship score engine and thresholds

### Wave 4: Policy And Hardening
- self-attested 18+ gate
- hosted-safe-mode fallback
- audit events
- observability and retry handling

## API And Job Surface Expectations
The implementation should align with a companion contract draft that includes:
- guest session bootstrap
- character listing and detail retrieval
- Facebook auth exchange
- conversation load and restore
- message send
- age-confirmation state set
- delayed reply execution jobs
- memory-update jobs
- relationship-update jobs

## Testing Expectations
### Required Test Areas
- guest 10-input cap enforcement
- guest reset on login
- Facebook-authenticated conversation restore
- one-user-one-character memory isolation
- delayed reply scheduling and delivery
- relationship score bounds and threshold behavior
- hard-guardrail refusal behavior
- hosted-safe-mode fallback behavior

### Recommended Test Layers
- module unit tests
- API integration tests
- worker job tests
- end-to-end browser tests for guest and authenticated flows

## Rollout Order
1. Internal seeded environment with Aira, Lyra, and Nova
2. Guest trial and authenticated chat smoke test
3. Delayed reply and memory validation pass
4. Safety and policy review pass
5. Cost and latency observation pass

## Open Questions
1. Can the phase 1 hosted model and provider policy support the intended adult-content policy for players who confirm they are 18+, or must adult sexual content wait for a later phase or local model?
2. What concrete retention durations should be assigned to guest-limited, account-durable, and audit-minimum lifecycle classes at implementation time?
3. The blueprint is stack-neutral because the repository does not yet define the concrete application framework for the client, backend, and worker.
4. Which session and secret-distribution mechanisms will be used in the chosen implementation stack?

## Recommendation Summary
Recommendation:
Implement the MVP as a modular monolith plus async worker with a stack-neutral module structure that preserves the architecture boundaries already approved. Keep the provider integration isolated, the memory model conversation-scoped, the guest-to-login reset rule enforced at both API and persistence levels, and execution aligned to the companion backlog and task artifact.