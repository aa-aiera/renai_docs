# Renai Game LLM MVP - Execution Backlog

## Document Control
- Status: Draft Execution Backlog
- Version: v1.0
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.0 | Major | Established the execution backlog as a managed implementation-planning artifact. | Implementor execution should use this backlog as the Planning v1.0 source of truth for milestone sequencing and progress updates. |

## Upstream Baseline
- Based On: MVP implementation blueprint v1.0 and architecture packet v1.0

## Executive Summary
This document applies the stack-neutral implementation blueprint to a sequenced execution backlog for engineering delivery. It is intended to be the working handoff plan for implementation readiness, milestone sequencing, and acceptance-driven delivery.

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
- milestone sequencing from decision lock through release readiness
- implementation tasks for client, backend, worker, data, policy, lifecycle, and security concerns
- dependency ordering and milestone exit criteria
- stack-neutral proposed file targets

This backlog does not cover:
- production code generation
- vendor-specific infrastructure implementation
- native mobile app delivery

## Assumptions
- the implementation remains stack-neutral until the team selects the concrete client, backend, and worker frameworks
- all implementation file paths below are proposed targets derived from the implementation blueprint
- phase 1 follows the current `hosted-safe-mode` planning assumption unless the ADR is revised

## Milestone Summary
| Milestone | Goal | Depends On | Exit Criteria |
| --- | --- | --- | --- |
| `M0` | Lock decisions and baseline runtime structure | none | stack mapping, lifecycle values, and session or secret approach are confirmed |
| `M1` | Establish foundation | `M0` | repo skeleton, core persistence, seed data, and discovery shell exist |
| `M2` | Deliver core chat loop | `M1` | guest and authenticated messaging round trip works with delayed replies |
| `M3` | Deliver continuity state | `M2` | memory, relationship state, and restore behavior work for one user-character pair |
| `M4` | Deliver policy, lifecycle, and security hardening | `M3` | policy gates, lifecycle enforcement, audit minimization, and rate controls are active |
| `M5` | Validate and prepare rollout | `M4` | automated checks, seeded environment smoke, and release checklist are complete |

## Milestone Backlog
### M0: Decision Lock And Runtime Baseline
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-001` | Lock implementation stack mapping | none | `apps/web-client/`, `services/backend-api/`, `services/worker/`, `infra/phase1/` | selected stack mapping exists for client, API, worker, datastore, and queue/cache without changing approved architecture boundaries |
| `IMP-002` | Confirm lifecycle and delete policy values | none | `infra/phase1/env-template/`, `services/backend-api/config/` | guest-limited, account-durable, and audit-minimum retention durations are defined and delete exposure policy is stated |
| `IMP-003` | Confirm session and secret-distribution approach | none | `services/backend-api/config/`, `services/worker/worker-entry/`, `infra/phase1/env-template/` | browser session mechanism and secret-distribution approach are selected and match the security architecture |

### M1: Foundation
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-101` | Scaffold repo structure and environment baseline | `IMP-001` | `apps/web-client/`, `services/backend-api/`, `services/worker/`, `infra/phase1/` | repo contains proposed top-level app, service, worker, test, and infra structure with environment template placeholders |
| `IMP-102` | Implement core persistence models and migrations | `IMP-001`, `IMP-002` | `services/backend-api/persistence/models/`, `services/backend-api/persistence/migrations/`, `services/backend-api/persistence/repositories/` | schema covers identity, conversations, messages, memory, relationship state, reply schedules, and audit entities with ownership constraints |
| `IMP-103` | Seed character catalog | `IMP-102` | `services/backend-api/modules/character-catalog/`, `tests/fixtures/` | Aira, Lyra, and Nova exist as seedable catalog entries aligned to the PRD |
| `IMP-104` | Build browser shell and discovery routes | `IMP-101`, `IMP-103` | `apps/web-client/app-shell/`, `apps/web-client/routes/home/`, `apps/web-client/routes/character-discovery/`, `apps/web-client/state/character-store/` | responsive discovery flow supports mobile swipe and desktop list entry points to chat selection |

### M2: Core Chat Loop
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-201` | Implement guest bootstrap and Facebook auth entrypoints | `IMP-101`, `IMP-102` | `services/backend-api/modules/identity/`, `services/backend-api/contracts/http/`, `apps/web-client/routes/auth/`, `apps/web-client/services/session-client/` | guest session create and Facebook auth exchange match the contract draft and preserve guest reset behavior |
| `IMP-202` | Implement conversations and messages API slice | `IMP-102`, `IMP-201` | `services/backend-api/modules/conversations/`, `services/backend-api/modules/messages/`, `services/backend-api/contracts/http/` | conversation resolve, load, and message send flows work with ownership checks |
| `IMP-203` | Implement chat UI and polling or refresh behavior | `IMP-104`, `IMP-202` | `apps/web-client/routes/chat/`, `apps/web-client/services/chat-client/`, `apps/web-client/state/conversation-store/`, `apps/web-client/components/delayed-message-status/` | chat screen can send user messages, show pending delay state, and render delivered character replies |
| `IMP-204` | Implement delayed reply scheduler and worker bootstrap | `IMP-202` | `services/worker/worker-entry/`, `services/worker/jobs/execute-delayed-reply/`, `services/backend-api/contracts/jobs/` | accepted user messages produce schedulable reply jobs and worker bootstrap can process due jobs |
| `IMP-205` | Implement provider adapter | `IMP-202`, `IMP-204` | `services/backend-api/modules/provider/` | provider boundary normalizes requests and responses without leaking provider specifics into other modules |

### M3: Continuity State
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-301` | Implement short-term context refresh flow | `IMP-202`, `IMP-204` | `services/backend-api/modules/memory/`, `services/worker/jobs/update-short-term-context/` | recent context is condensed and available to reply generation for the active conversation |
| `IMP-302` | Implement long-term memory jobs | `IMP-301`, `IMP-205` | `services/backend-api/modules/memory/`, `services/worker/jobs/update-long-term-memory/` | salient facts and milestones are written asynchronously and remain scoped to one user-character conversation |
| `IMP-303` | Implement relationship state engine | `IMP-202`, `IMP-205` | `services/backend-api/modules/relationship/`, `services/worker/jobs/update-relationship-state/` | deterministic score updates and threshold hints are persisted within defined bounds |
| `IMP-304` | Implement restore and continuity flow | `IMP-301`, `IMP-302`, `IMP-303` | `services/backend-api/modules/conversations/`, `apps/web-client/services/chat-client/`, `apps/web-client/routes/chat/` | returning authenticated users reopen the same character conversation with memory and relationship continuity |

### M4: Policy, Lifecycle, And Security Hardening
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-401` | Implement age confirmation and hosted-safe-mode policy gates | `IMP-205` | `services/backend-api/modules/policy-safety/`, `apps/web-client/components/age-confirmation-dialog/`, `apps/web-client/state/policy-store/` | policy checks enforce age confirmation, hard guardrails, and hosted-safe-mode behavior |
| `IMP-402` | Implement lifecycle enforcement | `IMP-102`, `IMP-204`, `IMP-304` | `services/backend-api/modules/conversations/`, `services/backend-api/modules/guest-trial/`, `services/worker/jobs/execute-delayed-reply/` | expired guest sessions and non-active conversations reject writes and do not receive delayed replies |
| `IMP-403` | Implement audit minimization and privileged trace boundaries | `IMP-102`, `IMP-401` | `services/backend-api/modules/audit/`, `services/worker/jobs/emit-audit-events/` | audit payloads are metadata-first and avoid becoming a shadow transcript store |
| `IMP-404` | Implement quota and abuse-resistance controls | `IMP-201`, `IMP-202` | `services/backend-api/modules/guest-trial/`, `services/backend-api/config/`, `services/backend-api/modules/audit/` | guest quota, rate controls, and suspicious access auditing are enforced server-side |

### M5: Validation And Rollout
| Task ID | Title | Depends On | Proposed Targets | Acceptance Criteria |
| --- | --- | --- | --- | --- |
| `IMP-501` | Implement automated test layers | `IMP-304`, `IMP-404` | `services/backend-api/tests/`, `services/worker/tests/`, `tests/integration/`, `tests/end-to-end/` | required guest, auth, delay, memory, relationship, policy, and lifecycle scenarios are covered |
| `IMP-502` | Run seeded environment smoke validation | `IMP-501` | `infra/phase1/docker-compose/`, `infra/phase1/app-config/`, `tests/fixtures/` | seeded environment validates discovery, chat, delayed replies, memory continuity, and policy enforcement |
| `IMP-503` | Prepare release checklist and observation baseline | `IMP-502` | `infra/phase1/env-template/`, `services/backend-api/config/`, `docs/03_implementation/` | release checklist exists for cost, latency, safety review, and rollback or disablement levers |

## Dependency Notes
- `IMP-001` to `IMP-003` are decision gates and should complete before implementation scaffolding starts.
- `IMP-204` and `IMP-205` together unlock the real delayed reply loop.
- `IMP-402` must be complete before rollout because lifecycle enforcement affects privacy, retention, and security guarantees.
- `IMP-501` must cover guest reset, lifecycle enforcement, and hosted-safe-mode behavior before seeded rollout.

## Acceptance Focus Areas
- guest reset is enforced at both session and persistence levels
- delayed replies are server-owned and lifecycle-aware
- memory never crosses character boundaries
- provider access remains server-side only
- audit traces remain minimized and operational rather than transcript-oriented

## Open Questions
1. Which concrete implementation stack will be chosen for the web client, backend API, worker, datastore, and queue/cache layer?
2. What exact retention durations should be assigned to guest-limited, account-durable, and audit-minimum classes?
3. Will phase 1 expose any user-facing delete capability, or remain internal/admin only?
4. Which concrete session mechanism and secret-distribution mechanism will the chosen stack use?

## Recommendation Summary
Recommendation:
Use this execution backlog as the human-readable delivery plan for the phase 1 MVP. Treat `M0` as a real gate, sequence the work by milestone dependency rather than by team convenience, and keep lifecycle, policy, and security tasks on the critical path rather than deferring them until after the chat loop works.