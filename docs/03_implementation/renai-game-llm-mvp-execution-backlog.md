# Renai Game LLM MVP - Execution Backlog

## Document Control
- Status: Draft Execution Backlog
- Version: v1.2.2
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.2.2 | Minor | Replaced the remaining directory-level M0 and M1 planning language with exact scaffold targets, naming conventions, and env-template expectations derived from the strict implementation blueprint. | Implementors can start repository scaffolding and env-template creation directly from the backlog without another interpretation pass. |
| 2026-04-26 | v1.2.1 | Minor | Locked the framework-level implementation choices inside the selected stack and replaced open M0 tooling and session questions with a concrete React, Go, Redis-backed cookie-session, and managed-secret baseline. | Implementors can start scaffolding against the selected toolchain without waiting for another planning decision round. |
| 2026-04-26 | v1.2.0 | Major | Locked the phase 1 stack baseline to React, Go, PostgreSQL, Redis, and RabbitMQ and updated M0 planning language to move from open stack choice to framework-level mapping within that baseline. | The task artifact and implementation kickoff work must preserve the selected stack while finalizing framework and session details. |
| 2026-04-26 | v1.1.0 | Major | Reconciled the backlog to PRD v3.0.2 and architecture v1.1.0 by removing guest-mode work, adopting Player and MC terminology, and sequencing authenticated-only chat, non-explicit policy, and internal lifecycle delete work. | The task artifact and implementation execution should follow this backlog as the current human-readable planning baseline. |
| 2026-04-26 | v1.0.0 | Major | Established the execution backlog as a managed implementation-planning artifact. | Implementor execution should use this backlog as the Planning v1.0.0 source of truth for milestone sequencing and progress updates. |

## Upstream Baseline
- Based On: MVP implementation blueprint v1.2.2 and architecture packet v1.1.0

## Executive Summary
This document applies the selected-stack implementation blueprint to a sequenced execution backlog for engineering delivery. It is intended to be the working handoff plan for implementation readiness, milestone sequencing, and acceptance-driven delivery.

All implementation targets below remain proposed because the repository does not yet contain the production application structure.

## Source Notes
- `docs/03_implementation/renai-game-llm-mvp-implementation-blueprint.md`
- `docs/03_implementation/renai-game-llm-mvp-api-contract-drafts.md`
- `docs/02_architecture/renai-game-llm-mvp-architecture-overview.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-erd.md`
- `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`
- `docs/02_architecture/renai-game-llm-mvp-security-architecture.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`

## Scope
This backlog covers:
- milestone sequencing from planning lock through release readiness
- implementation tasks for client, backend, worker, data, policy, lifecycle, and security concerns
- dependency ordering and milestone exit criteria
- proposed file targets mapped to the selected stack baseline

This backlog does not cover:
- production code generation
- vendor-specific infrastructure implementation
- native mobile app delivery

## Assumptions
- the phase 1 technology baseline is React for the browser client, Go for backend API and worker services, PostgreSQL for relational storage, Redis for cache, and RabbitMQ for queue delivery
- all implementation file paths below are proposed targets derived from the implementation blueprint
- discovery may remain browse-only, but all chat creation, restore, and send operations require an authenticated Player session

## Concrete Stack Baseline
| Layer | Selected Technology |
| --- | --- |
| Browser client | React |
| Backend API | Go |
| Async worker | Go |
| Relational database | PostgreSQL |
| Cache | Redis |
| Queue | RabbitMQ |

## Selected Framework And Security Baseline
| Concern | Selected Choice |
| --- | --- |
| Browser toolchain | React + TypeScript + Vite |
| Routing | React Router |
| Server-state fetching | TanStack Query |
| Local UI state | Zustand |
| Go router | `chi` on `net/http` |
| Request validation | `go-playground/validator` |
| PostgreSQL data access | `pgx` + `sqlc` |
| Migrations | `goose` |
| Redis client | `go-redis` |
| RabbitMQ client | `amqp091-go` |
| Logging | `log/slog` |
| Browser session transport | Secure `HttpOnly` opaque session cookie |
| Server-side session storage | Redis |
| Secret distribution | Managed secret store injected to backend and worker env vars |

## Initial Scaffold Targets
- `apps/web-client/package.json`, `vite.config.ts`, `tsconfig.json`, `vitest.config.ts`, `playwright.config.ts`, and `.env.example`
- `apps/web-client/src/app/router.tsx`, `providers.tsx`, and `queryClient.ts`
- `services/backend-api/go.mod`, `cmd/api/main.go`, `.env.example`, `sqlc.yaml`, and `db/migrations/0001_initial_schema.sql`
- `services/backend-api/internal/platform/*`, `internal/http/*`, and `internal/modules/*`
- `services/worker/go.mod`, `cmd/worker/main.go`, `.env.example`, and `internal/jobs/*`
- `infra/phase1/env-template/web-client.env.example`, `backend-api.env.example`, `worker.env.example`, and `shared.infra.env.example`

## Milestone Summary
| Milestone | Goal | Depends On | Exit Criteria |
| --- | --- | --- | --- |
| `M0` | Encode planning decisions and runtime structure | none | selected framework mapping, selected cookie-session and secret-distribution approach, exact scaffold targets, lifecycle constants, and internal delete operating model are encoded in scaffold and config conventions |
| `M1` | Establish foundation | `M0` | repo skeleton, core persistence, seed data, and discovery shell exist |
| `M2` | Deliver authenticated chat loop | `M1` | authenticated Player messaging round trip works with delayed replies |
| `M3` | Deliver continuity state | `M2` | memory, relationship state, and restore behavior work for one Player-MC pair |
| `M4` | Deliver policy, lifecycle, and security hardening | `M3` | non-explicit policy enforcement, lifecycle enforcement, audit minimization, and rate controls are active |
| `M5` | Validate and prepare rollout | `M4` | automated checks, seeded environment smoke, and release checklist are complete |

## Milestone Backlog
### M0: Decision Lock And Runtime Baseline
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-001` | Encode selected framework and library baseline | none | `apps/web-client/`, `services/backend-api/`, `services/worker/`, `infra/phase1/` | `apps/web-client/package.json`, `vite.config.ts`, `tsconfig.json`, `services/backend-api/go.mod`, `services/backend-api/sqlc.yaml`, and `services/worker/go.mod` encode the selected React, Go, PostgreSQL, Redis, and RabbitMQ library baseline without changing approved architecture boundaries |
| `IMP-002` | Encode lifecycle constants and admin-delete operating model | none | `infra/phase1/env-template/`, `services/backend-api/config/`, `services/backend-api/modules/admin-lifecycle/` | 12-month conversation retention, 30-day derived-state purge, and 90-day audit retention are encoded and the internal delete operating model is documented |
| `IMP-003` | Encode session and secret-distribution baseline | none | `services/backend-api/config/`, `services/worker/worker-entry/`, `infra/phase1/env-template/` | `infra/phase1/env-template/web-client.env.example`, `backend-api.env.example`, and `worker.env.example` define the secure `HttpOnly` cookie, Redis-backed session, and managed-secret injection baseline |

### M1: Foundation
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-101` | Scaffold repo structure and environment baseline | `IMP-001` | `apps/web-client/`, `services/backend-api/`, `services/worker/`, `infra/phase1/` | repo contains the exact scaffold files from the implementation blueprint and env-template placeholders for browser config, PostgreSQL, Redis, RabbitMQ, Facebook auth, and LLM provider settings |
| `IMP-102` | Implement core persistence models and migrations | `IMP-001`, `IMP-002` | `services/backend-api/persistence/models/`, `services/backend-api/persistence/migrations/`, `services/backend-api/persistence/repositories/` | schema covers identity, conversations, messages, memory, relationship state, reply schedules, and audit entities with Player-MC ownership constraints |
| `IMP-103` | Seed MC catalog | `IMP-102` | `services/backend-api/modules/mc-catalog/`, `tests/fixtures/` | Aira, Lyra, and Nova exist as seedable MC entries aligned to the PRD |
| `IMP-104` | Build browser shell, discovery routes, and login entry | `IMP-101`, `IMP-103` | `apps/web-client/app-shell/`, `apps/web-client/routes/home/`, `apps/web-client/routes/mc-discovery/`, `apps/web-client/routes/auth/`, `apps/web-client/state/mc-store/` | responsive discovery flow supports mobile swipe and desktop list entry points and can route authenticated Players into chat |

### M2: Authenticated Chat Loop
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-201` | Implement session inspection and Facebook auth entrypoints | `IMP-101`, `IMP-102` | `services/backend-api/modules/identity/`, `services/backend-api/contracts/http/`, `apps/web-client/routes/auth/`, `apps/web-client/services/session-client/`, `apps/web-client/services/auth-client/` | session inspection and Facebook auth exchange match the contract draft and resolve to internal Player identity |
| `IMP-202` | Implement conversations and messages API slice | `IMP-102`, `IMP-201` | `services/backend-api/modules/conversations/`, `services/backend-api/modules/messages/`, `services/backend-api/contracts/http/` | conversation resolve, load, and message send flows work with authenticated ownership checks |
| `IMP-203` | Implement authenticated chat UI and refresh behavior | `IMP-104`, `IMP-202` | `apps/web-client/routes/chat/`, `apps/web-client/services/chat-client/`, `apps/web-client/state/conversation-store/`, `apps/web-client/components/auth-required-gate/`, `apps/web-client/components/delayed-message-status/` | chat screen can send Player messages, show pending delay state, render delivered MC replies, and block unauthenticated chat access |
| `IMP-204` | Implement delayed reply scheduler and worker bootstrap | `IMP-202` | `services/worker/worker-entry/`, `services/worker/jobs/execute-delayed-reply/`, `services/backend-api/contracts/jobs/` | accepted Player messages produce schedulable reply jobs and worker bootstrap can process due jobs |
| `IMP-205` | Implement provider adapter and policy-mode hooks | `IMP-202`, `IMP-204` | `services/backend-api/modules/provider/`, `services/backend-api/modules/policy-safety/` | provider boundary normalizes requests and responses and can run the approved non-explicit phase 1 mode |

### M3: Continuity State
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-301` | Implement short-term context refresh flow | `IMP-202`, `IMP-204` | `services/backend-api/modules/memory/`, `services/worker/jobs/update-short-term-context/` | recent context is condensed and available to reply generation for the active conversation |
| `IMP-302` | Implement long-term memory jobs | `IMP-301`, `IMP-205` | `services/backend-api/modules/memory/`, `services/worker/jobs/update-long-term-memory/` | salient facts and milestones are written asynchronously and remain scoped to one Player-MC conversation |
| `IMP-303` | Implement relationship state engine | `IMP-202`, `IMP-205` | `services/backend-api/modules/relationship/`, `services/worker/jobs/update-relationship-state/` | deterministic score updates and threshold hints are persisted within defined bounds |
| `IMP-304` | Implement restore and continuity flow | `IMP-301`, `IMP-302`, `IMP-303` | `services/backend-api/modules/conversations/`, `apps/web-client/services/chat-client/`, `apps/web-client/routes/chat/`, `apps/web-client/components/conversation-list/` | returning authenticated Players reopen the same MC conversation with memory and relationship continuity |

### M4: Policy, Lifecycle, And Security Hardening
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-401` | Implement hosted-model non-explicit policy and hard guardrails | `IMP-205` | `services/backend-api/modules/policy-safety/`, `services/backend-api/modules/audit/` | policy checks enforce hard guardrails and the hosted-model non-explicit phase 1 posture |
| `IMP-402` | Implement lifecycle enforcement and admin-managed delete workflow | `IMP-102`, `IMP-204`, `IMP-304` | `services/backend-api/modules/conversations/`, `services/backend-api/modules/admin-lifecycle/`, `services/worker/jobs/execute-delayed-reply/`, `services/worker/jobs/purge-conversation/` | deleted or purge-pending conversations reject reads and writes in normal Player flows and purge completes for owner-bound data within the approved window |
| `IMP-403` | Implement audit minimization and privileged trace boundaries | `IMP-102`, `IMP-401` | `services/backend-api/modules/audit/`, `services/worker/jobs/emit-audit-events/` | audit payloads are metadata-first and avoid becoming a shadow transcript store |
| `IMP-404` | Implement auth rate limiting and abuse-resistance controls | `IMP-201`, `IMP-202` | `services/backend-api/config/`, `services/backend-api/modules/audit/`, `services/backend-api/modules/identity/` | rate controls, suspicious-access tracing, and ownership-sensitive audit hooks are enforced server-side |

### M5: Validation And Rollout
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-501` | Implement automated test layers | `IMP-304`, `IMP-404` | `services/backend-api/tests/`, `services/worker/tests/`, `tests/integration/`, `tests/end-to-end/` | required auth, delay, memory, relationship, policy, lifecycle, and delete scenarios are covered |
| `IMP-502` | Run seeded environment smoke validation | `IMP-501` | `infra/phase1/docker-compose/`, `infra/phase1/app-config/`, `tests/fixtures/` | seeded environment validates discovery, login, chat, delayed replies, memory continuity, and policy enforcement |
| `IMP-503` | Prepare release checklist and observation baseline | `IMP-502` | `infra/phase1/env-template/`, `services/backend-api/config/`, `docs/03_implementation/` | release checklist exists for cost, latency, safety, and rollback review and observation baseline covers provider errors, delayed replies, safety refusals, and lifecycle events |

## Dependency Notes
- `IMP-001` to `IMP-003` are planning gates and should complete before implementation scaffolding starts.
- `IMP-204` and `IMP-205` together unlock the real delayed reply loop.
- `IMP-402` must be complete before rollout because lifecycle enforcement affects privacy, retention, and security guarantees.
- `IMP-501` must cover authenticated access control, lifecycle enforcement, and hosted-model non-explicit behavior before seeded rollout.

## Acceptance Focus Areas
- authentication is enforced before chat creation, restore, and send
- delayed replies are server-owned and lifecycle-aware
- memory never crosses MC boundaries
- provider access remains server-side only
- hosted-model output stays within the approved non-explicit phase 1 posture
- audit traces remain minimized and operational rather than transcript-oriented
- internal delete and purge behavior align to the approved retention windows

## Open Questions
No open questions are currently blocking the execution backlog baseline.

## Recommendation Summary
Recommendation:
Use this execution backlog as the human-readable delivery plan for the phase 1 MVP. Treat `M0` as the point where exact scaffold files and env templates are encoded, then move directly into `M1` scaffolding without reopening framework or session decisions.
