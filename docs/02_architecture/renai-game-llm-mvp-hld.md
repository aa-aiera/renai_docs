# Renai Game LLM MVP - High-Level Design

## Document Control
- Status: Draft HLD
- Version: v1.1.0
- Last Updated: 2026-04-26
- Owner: SA

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.1.0 | Major | Reconciled the HLD to PRD v3.0.2 by removing guest mode, adopting Player and MC terminology, enforcing login-required chat, and locking in the hosted-model content, retention, and deletion baseline. | ERD, sequence flows, component diagrams, and implementation-planning artifacts must align to the authenticated-only chat model and fixed lifecycle policy. |
| 2026-04-26 | v1.0.0 | Major | Established the HLD as the managed high-level architecture baseline for the MVP. | Technical Lead planning artifacts should continue to align to HLD v1.0.0 until the architecture packet is revised. |

## Upstream Baseline
- Based On: Phase 1 MVP PRD v3.0.2

## Executive Summary
This document defines the high-level system architecture for the phase 1 MVP of the renai-game-style LLM chat product. The recommended architecture remains a modular monolith plus an async worker, but the current baseline now assumes authenticated Player chat only, one durable Player-MC conversation root per pair, hosted-model non-explicit romance only, and explicit lifecycle rules for retention and admin-managed deletion.

The system is optimized for:
- a responsive browser MVP
- 3 launch MCs
- Facebook-based Player identity in phase 1
- delayed, human-like chat behavior
- conversation-scoped short-term and long-term memory
- deterministic relationship-state updates
- minimized audit traces and fixed retention windows

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`
- `docs/02_architecture/renai-game-llm-mvp-architecture-overview.md`

## Problem Statement
The product must deliver a dating-app-style AI companion experience without relying on dialogue trees. The system must preserve conversational continuity, relationship progression, MC consistency, and human-like pacing while staying within hosted-model constraints, the MVP budget cap, and the approved privacy and deletion posture.

## In Scope
- Responsive browser client for desktop and mobile
- Facebook login for Player identity
- MC discovery and selection
- One-on-one Player-to-MC chat
- Delayed replies up to 5 minutes
- Conversation-scoped short-term and long-term memory
- Relationship-state computation and hidden phase 1 metrics
- Hosted-model non-explicit romance and friendship progression
- Admin-managed deletion and lifecycle enforcement

## Out Of Scope
- Guest or anonymous chat sessions
- Explicit adult sexual content in phase 1
- Player-facing self-service delete controls
- Shared-world memory
- Cross-MC memory sharing
- Native mobile apps

## Architecture Goals
- Keep chat continuity scoped to one Player-MC pair.
- Make delayed reply execution backend-owned and lifecycle-aware.
- Isolate provider-specific logic behind one adapter layer.
- Keep policy enforcement explicit so hosted-model limits do not leak into UI behavior unpredictably.
- Keep derived memory and relationship state owner-bound so deletion propagates cleanly.

## System Shape
```text
Responsive Browser Client
  -> Backend API / BFF
     -> Relational Database
     -> Queue / Cache
     -> Hosted LLM Provider (via adapter)
  -> Async Worker
     -> Queue / Cache
     -> Relational Database
     -> Hosted LLM Provider (via adapter)
```

## Core Modules
| Module | Responsibility |
| --- | --- |
| Identity And Session Module | Resolve Facebook auth into Player identity and browser session state |
| MC Catalog Module | Serve MC discovery and profile metadata |
| Conversations Module | Create, load, restore, and lifecycle-check Player-MC conversations |
| Messages Module | Persist Player and MC messages and load transcript slices |
| Chat Orchestration Module | Coordinate policy, memory, relationship, and delayed scheduling on message send |
| Memory Module | Load prompt context and write short-term or long-term memory artifacts |
| Relationship Engine | Compute and store relationship-state changes and tone hints |
| Policy And Safety Module | Enforce hard restrictions and hosted-model non-explicit posture |
| Provider Adapter | Normalize hosted-provider requests and responses |
| Audit Module | Persist metadata-first operational and safety traces |
| Async Worker | Execute delayed replies, derived-state updates, and purge jobs |

## Runtime Decisions
### 1. Chat Requires Authentication
Chat creation, restore, and message send operations require an authenticated Player session. Discovery may be public, but the conversation boundary is not.

### 2. One Durable Conversation Root Per Player-MC Pair
The primary continuity boundary is one durable conversation root for one Player and one MC. All derived memory and relationship state remain anchored to that root.

### 3. Delayed Replies Are Asynchronous
The backend accepts a Player message, persists it, computes reply timing, and schedules worker execution. The worker revalidates lifecycle state before generating or delivering the MC reply.

### 4. Hosted Public Model Is Non-Explicit In Phase 1
Phase 1 supports romance and friendship progression but not explicit adult sexual content. The policy layer constrains outputs accordingly.

### 5. Lifecycle Rules Are Fixed In Phase 1
- conversation history retention: 12 months after last activity unless earlier admin-managed deletion is triggered
- derived memory and relationship state: hard-delete within 30 days after confirmed deletion
- audit-minimum traces: 90 days with metadata-first payload design

### 6. No Player-Facing Delete In Phase 1
Deletion is an internal or admin-managed workflow. The user-facing system must respect deleted, purge-pending, and inaccessible states but does not expose a self-service delete feature.

## Data Model Summary
| Entity Group | Purpose |
| --- | --- |
| Player identity | Authenticated Player account and provider links |
| MC catalog | Curated MC roster and behavior metadata |
| Conversation root | Ownership, lifecycle, and continuity anchor for one Player-MC pair |
| Messages | Append-only transcript entries for Player, MC, and system events |
| Short-term context | Compact recent-context text for prompt assembly |
| Long-term memory | Durable prompt-ready memory artifacts scoped to one conversation |
| Relationship state | Structured hidden metrics for one conversation |
| Reply schedule | Delayed reply execution state |
| Audit trace | Metadata-first operational and safety events |

## Lifecycle And Retention Architecture
### Conversation Lifecycle States
- `active`: normal read and write access
- `soft-deleted`: inaccessible to normal runtime flows, pending purge orchestration
- `purge-pending`: queued for irreversible cleanup
- `purged`: no longer available in the product data path

### Ownership Rules
- `Conversation` is the deletion root for messages, short-term context, long-term memory, relationship state, and reply schedules.
- Audit traces remain separate from the Player-visible transcript store.
- Derived state never survives the deletion of its owning conversation.

### Retention Rules
- Conversation history: 12 months after `last_activity_at`
- Derived memory and relationship state: hard-delete within 30 days after confirmed deletion
- Audit-minimum traces: 90 days

## Core Runtime Journeys
### 1. Player Login And MC Chat Entry
1. Browser authenticates through Facebook.
2. Backend resolves or creates the Player account.
3. Player browses or selects an MC.
4. Backend resolves the Player-MC conversation root.
5. Browser enters the chat screen.

### 2. Player Message Send
1. Browser submits a Player message.
2. Backend validates session ownership and lifecycle state.
3. Backend loads MC profile, memory, relationship state, and policy context.
4. Backend stores the Player message and creates a reply schedule.
5. Browser receives accepted state and waits for delayed reply delivery.

### 3. Worker Reply Execution
1. Worker fetches a due reply job.
2. Worker rechecks conversation lifecycle state.
3. Worker assembles context and applies hosted-model non-explicit policy.
4. Worker calls the provider through the adapter.
5. Worker persists the MC reply and updates the schedule state.

### 4. Derived-State Update
1. Worker updates short-term context.
2. Worker writes long-term memory when salience rules pass.
3. Worker updates relationship state.
4. Worker emits metadata-first audit traces.

### 5. Admin-Managed Delete
1. Internal or admin workflow marks the conversation `soft-deleted`.
2. Runtime access and delayed replies stop immediately.
3. Background purge marks the conversation `purge-pending`.
4. Owner-bound derived artifacts are hard-deleted within 30 days.
5. Audit traces continue only under their separate 90-day retention rule.

## Policy And Safety Architecture
### Safety Rules
- MCs do not make important life decisions for Players.
- MCs do not persuade or coach illegal acts.
- MCs do not act as serious real-world problem solvers.
- Non-explicit romance and friendship are allowed.
- Explicit adult sexual content is not allowed in phase 1.

### Hosted-Model Policy Enforcement
- Pre-generation rules set the allowed response mode.
- Post-generation validation blocks or rewrites disallowed output.
- Provider behavior is treated as nondeterministic and must not be trusted on prompt design alone.

## Security And Operations Constraints
- No direct browser-to-provider access.
- Session and ownership checks happen server-side on every conversation write and restore operation.
- Purge workers must be idempotent.
- Audit traces must remain metadata-first and access-controlled.
- Provider credentials remain server-side only.

## Risks
- Hosted-model tone may still drift toward undesired explicitness without strong policy checks.
- Delayed reply jobs can create stale execution attempts unless lifecycle checks run again in the worker.
- Over-retention risk remains if audit payloads become too transcript-like.
- Relationship progression may feel uneven if memory and score updates drift apart operationally.

## Recommendation Summary
Recommendation:
Adopt the modular monolith plus async worker architecture in phase 1, keep memory strictly scoped per Player-MC pair, enforce hosted-model non-explicit behavior as a first-class policy rule, and treat conversation ownership plus fixed retention windows as the core lifecycle boundary for all derived state.
