# Renai Game LLM MVP - Architecture Overview And Index

## Document Control
- Status: Draft Architecture Overview
- Version: v1.1.0
- Last Updated: 2026-04-26
- Owner: SA

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.1.0 | Major | Reconciled the architecture overview to PRD v3.0.2 by removing guest mode, adopting Player and MC terminology, and locking in the hosted-model content, retention, and deletion policy baseline. | Architecture companion documents and implementation-planning artifacts must align to the same login-required chat flow and policy baseline. |
| 2026-04-26 | v1.0.0 | Major | Established the architecture overview as a managed entry point for the current architecture packet. | Technical Lead and Implementor should use Architecture v1.0.0 as the active packet baseline until a later architecture revision is published. |

## Upstream Baseline
- Based On: Phase 1 MVP PRD v3.0.2 and complete architecture packet v1.1.0

## Executive Summary
This document is the entry point for the renai-game-style LLM MVP architecture set. It summarizes the current system direction, identifies the source-of-truth documents for each architectural concern, and gives engineering, product, and review stakeholders a fast way to navigate the packet.

The architecture baseline is now consistent around these decisions:
- responsive browser MVP in phase 1
- modular monolith backend plus async worker
- relational database plus queue or cache layer
- Facebook-based Player identity with login required before chat
- one-on-one conversation scope only
- per-Player-per-MC memory isolation
- hidden phase 1 relationship scores
- provider abstraction for future local-LLM migration
- lifecycle-aware privacy and retention model
- non-explicit hosted-model phase 1 with adult sexual content deferred to a later local-model phase

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-erd.md`
- `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md`
- `docs/02_architecture/renai-game-llm-mvp-component-diagrams.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`
- `docs/02_architecture/renai-game-llm-mvp-security-architecture.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`
- `docs/03_implementation/renai-game-llm-mvp-implementation-blueprint.md`
- `docs/03_implementation/renai-game-llm-mvp-api-contract-drafts.md`

## Problem Statement
The document set has grown from a single PRD into a full architecture and engineering handoff packet. Without a concise overview, readers must reconstruct the document map themselves and may miss which file is authoritative for memory scope, security boundaries, privacy lifecycle rules, or provider-policy assumptions.

## Architecture Baseline
### Product And Runtime Shape
- one responsive browser client for desktop and mobile use
- one backend API or BFF surface
- one async worker role for delayed replies and background jobs
- one relational system of record
- one queue or cache layer for scheduling and transient coordination

### Behavior And Policy Shape
- open-ended LLM chat instead of scripted route trees
- delayed reply scheduling owned by the backend
- authenticated Player login required before chat can start or continue
- romance and friendship both supported
- hosted public model phase 1 remains non-explicit for all Players
- adult sexual content is deferred to a later local-model phase and needs separate approval before release

### Data And State Shape
- one durable conversation root per Player-MC pair in authenticated use
- no shared-world memory
- no cross-MC memory
- derived memory and relationship state follow conversation ownership
- conversation data retained for 12 months after last activity unless earlier admin-managed deletion is triggered
- derived memory and relationship artifacts hard-deleted within 30 days after confirmed deletion
- audit-minimum traces retained for 90 days with minimized metadata-only scope by default

## Document Map
| Document | Primary Purpose | When To Use It |
| --- | --- | --- |
| `docs/01_requirements/renai-game-llm-prd.md` | Product scope, requirements, and acceptance criteria | Start here for product intent and approved MVP boundaries |
| `docs/02_architecture/renai-game-llm-mvp-hld.md` | Core system architecture and deployment model | Use for top-level technical design |
| `docs/02_architecture/renai-game-llm-mvp-erd.md` | Conceptual data model and entity boundaries | Use for schema and persistence planning |
| `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md` | Runtime request and job flows | Use for execution order and coordination behavior |
| `docs/02_architecture/renai-game-llm-mvp-component-diagrams.md` | Static visual architecture views | Use for diagram-first explanation and review |
| `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md` | Data lifecycle, retention, and deletion model | Use for lifecycle and deletion policy design |
| `docs/02_architecture/renai-game-llm-mvp-security-architecture.md` | Trust boundaries, auth, secrets, and abuse-resistance architecture | Use for security review and hardening planning |
| `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md` | Hosted-model content posture decision record | Use when discussing the non-explicit phase 1 policy and later local-model content expansion |
| `docs/03_implementation/renai-game-llm-mvp-implementation-blueprint.md` | File-by-file implementation planning | Use for engineering execution planning |
| `docs/03_implementation/renai-game-llm-mvp-api-contract-drafts.md` | Draft HTTP and job contracts | Use for interface and integration planning |

## Recommended Reading Order
1. PRD
2. HLD
3. ERD
4. Sequence Flows
5. Component Diagrams
6. Privacy, Retention, And Data Lifecycle Architecture
7. Security Architecture
8. ADR-0001 for hosted-model content posture
9. Implementation Blueprint
10. API Contract Drafts

## Decision Traceability Summary
| Decision Area | Current Direction | Source Of Truth |
| --- | --- | --- |
| Client shape | Responsive browser MVP | PRD, HLD |
| Runtime architecture | Modular monolith plus async worker | HLD |
| Identity model | Facebook auth with login-required chat | PRD, HLD, Sequence Flows |
| Memory scope | Per-Player-per-MC only | PRD, HLD, ERD |
| Relationship model | Deterministic structured metrics with hidden phase 1 scores | PRD, HLD |
| Delay behavior | Backend-owned delayed replies up to 5 minutes | PRD, HLD, Sequence Flows |
| Provider portability | Provider adapter with future local-LLM path | HLD, Component Diagrams |
| Privacy lifecycle | Conversation-root ownership with fixed phase 1 retention windows and admin-managed deletion | Privacy And Retention Architecture |
| Security boundary | No direct browser-to-provider access; server-side trust boundaries | Security Architecture |
| Hosted-model content posture | Non-explicit phase 1, explicit adult content deferred to later local-model phase | ADR-0001 |

## Current Packet Strengths
- the system boundary, runtime shape, and data model are aligned
- login-required chat and per-MC memory isolation are explicit in the current PRD baseline
- delayed reply behavior is modeled both statically and dynamically
- lifecycle and deletion architecture now exists as a dedicated companion
- implementation-facing blueprint and API drafts are already available downstream

## Remaining Sync Work
1. Update the architecture companions to remove guest-mode data paths, guest-trial control points, and stale user/character wording.
2. Update the planning packet to remove guest-session implementation scope and replace open policy questions with the approved retention and deletion values.
3. Keep audit trace payload scope minimal and metadata-first unless a later policy change explicitly broadens it.

## Recommendation Summary
Recommendation:
Use this overview as the front door to the architecture packet. Treat the PRD, HLD, ERD, sequence flows, component diagrams, privacy lifecycle companion, security companion, and ADR as the current phase 1 architecture source set, then use the implementation blueprint and API contracts as downstream engineering handoff artifacts.