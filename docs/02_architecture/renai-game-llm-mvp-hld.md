# Renai Game LLM MVP - High-Level Design

## Document Control
- Status: Draft HLD
- Version: v1.0
- Last Updated: 2026-04-26
- Owner: SA

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.0 | Major | Established the HLD as the managed high-level architecture baseline for the MVP. | Technical Lead planning artifacts should continue to align to HLD v1.0 until the architecture packet is revised. |

## Upstream Baseline
- Based On: Phase 1 MVP PRD v1.0

## Executive Summary
This document defines the high-level system architecture for the phase 1 MVP of the renai-game-style LLM chat product. The recommended design is a modular monolith with an asynchronous reply scheduler, a provider abstraction layer for hosted LLM access, isolated per-user-per-character memory, a deterministic relationship-state engine, and a policy layer that constrains unsafe or disallowed output.

The architecture is optimized for:
- a responsive browser MVP
- 3 launch characters
- Facebook-based returning identity plus a guest trial path
- delayed, human-like chat behavior
- short-term and long-term memory per user-character pair
- future migration from a hosted public LLM to a local LLM
- operation on a medium-sized instance in phase 1

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`

## Problem Statement
The product must deliver a dating-app-style AI companion experience without relying on rigid dialogue trees. The system must preserve conversational continuity, relationship progression, and character consistency while remaining safe, explainable enough to tune, and affordable enough for an MVP.

The architecture must also support:
- guest trial usage with a hard 10-input cap
- authenticated returning users through Facebook login
- one-on-one memory isolation per user-character pair
- rule-governed relationship evolution
- asynchronous delayed replies up to 5 minutes
- future provider replacement with minimal product redesign

## Goals
- Provide a stable phase 1 architecture that supports the full MVP feature set.
- Keep phase 1 deployable on a medium-sized instance.
- Avoid premature microservice complexity.
- Preserve a clean path to future local-LLM adoption.
- Keep the memory, scoring, and safety behaviors inspectable and tunable.

## Non-Goals
- Multi-character chat rooms or shared world memory.
- Cross-character memory sharing.
- Native mobile apps in phase 1.
- A fully distributed microservice platform in phase 1.
- Premium score visibility in phase 1.

## Constraints And Assumptions
### Confirmed Constraints
- Phase 1 client is a responsive browser application.
- Returning identity starts with Facebook login.
- Guest users can chat before login but are limited to 10 user input messages.
- Guest state resets on login and does not transfer into the authenticated account.
- Memory is isolated to one user-character pair.
- Relationship scores are maintained internally in phase 1.
- Hosted public LLM is used in phase 1.
- Local LLM migration is planned for phase 2.
- Adult sexual content is gated in phase 1 by a simple self-attested 18+ confirmation dialog.

### Assumptions
- One active conversation context exists per user-character pair at a time.
- Character metadata is curated by product/admin workflows outside the MVP player-facing system.
- The phase 1 backend can run multiple containers on one medium-sized host.

## Architecture Options
### Option A: Thin Backend With Direct Provider Calls
Description:
The web application talks to a minimal backend that forwards prompts directly to the hosted provider and stores chat history.

Pros:
- Fastest initial build
- Lowest architectural overhead

Cons:
- Weak separation between product logic and provider logic
- Harder to migrate to a local LLM later
- Delayed replies, memory summarization, and relationship scoring become tangled in request handlers
- Poor foundation for safety and policy controls

Assessment:
Rejected. This option is too brittle for the memory, scoring, and policy requirements.

### Option B: Modular Monolith With Async Worker And Provider Abstraction
Description:
A single backend codebase contains separate modules for identity, chat orchestration, memory, scoring, safety, and provider access. A worker handles delayed replies and background memory tasks.

Pros:
- Fits the MVP deployment constraint
- Clean enough to scale later
- Supports delayed responses and background jobs naturally
- Keeps LLM provider coupling isolated
- Easier to test and tune than a thin backend

Cons:
- More design work than a thin backend
- Requires queue or scheduling support

Assessment:
Recommended. This provides the best balance of MVP speed, system clarity, and future extensibility.

### Option C: Event-Driven Microservices From Day One
Description:
Identity, chat, memory, scoring, scheduling, safety, and analytics all run as separate services with an event bus.

Pros:
- Maximum long-term decomposition
- Clear service boundaries at scale

Cons:
- Excessive operational overhead for phase 1
- Higher latency and debugging complexity
- Misaligned with the medium-instance MVP goal

Assessment:
Rejected for phase 1. This is premature for the current scope.

## Recommended Architecture
Recommendation:
Use a modular monolith backend plus an asynchronous worker, backed by a relational datastore and an in-memory queue/cache layer.

### Logical Component View
```text
Responsive Web Client
    |
    v
API Layer / BFF
    |
    +--> Identity Module
    +--> Character Catalog Module
    +--> Chat Orchestrator
    +--> Guest Trial Gate
    +--> Policy and Safety Module
    +--> Relationship Engine
    +--> Memory Module
    +--> Provider Adapter
    |
    +--> Relational Data Store
    +--> In-Memory Queue / Cache
    +--> Object or Blob Store (optional, non-critical for MVP)

Async Worker
    |
    +--> Delayed Reply Scheduler
    +--> Memory Summarization Jobs
    +--> Post-Reply Score Update Jobs
    +--> Observability / Audit Events
```

### Component Responsibilities
| Component | Responsibility |
| --- | --- |
| Responsive Web Client | Character browsing, chat UI, guest/login prompts, age confirmation dialog, delayed-message display |
| API Layer / BFF | Single client-facing backend surface for browser requests |
| Identity Module | Guest session handling, Facebook auth, account/session binding, future multi-provider linking |
| Guest Trial Gate | Enforces 10 input-message limit for non-logged-in users |
| Character Catalog Module | Stores character profiles, persona prompts, limits, and profile metadata |
| Chat Orchestrator | Coordinates the end-to-end message pipeline |
| Memory Module | Manages recent context, long-term summaries, and prompt-ready memory assembly |
| Relationship Engine | Stores and updates metrics such as friendship, trust, attraction, comfort, and hidden system scores |
| Policy and Safety Module | Applies age gating, unsafe-role restrictions, topic constraints, and output guardrails |
| Provider Adapter | Normalizes requests and responses across hosted and future local LLMs |
| Async Worker | Handles delayed reply execution and background processing |
| Relational Data Store | Persists users, sessions, characters, chats, messages, memories, and scores |
| In-Memory Queue / Cache | Supports scheduled replies, temporary state, and low-latency job dispatch |

## Core Architectural Decisions
### 1. Modular Monolith Over Microservices
Reason:
The MVP must run on a medium-sized instance and should not incur distributed-system overhead before usage justifies it.

### 2. Async Reply Scheduling Is First-Class
Reason:
The product requires normal reply pacing and sometimes much longer low-interest delays. This should not be simulated only in the frontend. The backend must own the authoritative reply schedule.

### 3. Memory Is Strictly Scoped To User-Character Pairs
Reason:
This matches the PRD and keeps the MVP explainable, private by default, and simpler to tune.

### 4. Relationship Scoring Uses A Hybrid Model
Reason:
The architecture should support mostly deterministic score updates, optionally informed by structured LLM signals, so progression remains tunable and less arbitrary.

### 5. Provider Abstraction Is Mandatory In Phase 1
Reason:
The system must switch from hosted LLMs to local LLMs in phase 2 without redesigning the product-facing workflow.

## Data Architecture
### Primary Domain Entities
| Entity | Purpose |
| --- | --- |
| UserAccount | Persistent authenticated player identity |
| GuestSession | Temporary anonymous session used before login |
| IdentityLink | Future mapping for Facebook, LINE, email, and other auth providers |
| CharacterProfile | Public profile plus persona, interests, boundaries, and age metadata |
| Conversation | One user-character pair conversation root |
| Message | One user or character utterance |
| ShortTermContext | Recent conversation window or condensed rolling context |
| LongTermMemory | Persistent text facts or summaries for a user-character pair |
| RelationshipState | Score record for one user-character pair |
| ReplySchedule | Pending delayed reply job metadata |
| PolicyState | Age-confirmation state and other gating flags |
| AuditEvent | Safety, moderation, and system behavior trace events |

### Recommended Storage Boundaries
| Data Type | Recommended Store Type |
| --- | --- |
| Users, characters, conversations, messages, scores | Relational database |
| Scheduled reply jobs, transient counters, rate limits | In-memory queue/cache |
| Memory summaries and prompt snapshots | Relational database initially; blob/object store optional later |

### Memory Scope Rule
All memory records are keyed by:
- `player_identity`
- `character_id`

No shared-world memory and no cross-character memory records exist in phase 1.

## Privacy, Retention, And Deletion Architecture
### Lifecycle Model
The MVP should use lifecycle classes and ownership boundaries rather than hardcoded retention durations at the architecture level.

Recommended retention classes:
- ephemeral operational data
- guest-limited data
- account-durable conversation data
- audit-minimum trace data

### Core Lifecycle Rules
- guest-owned conversations are disposable, do not transfer on login, and become eligible for expiry and purge after the guest lifecycle ends
- authenticated conversation continuity is durable by default but remains owner-bound
- `ShortTermContext`, `LongTermMemory`, and `RelationshipState` must follow the lifecycle of the owning `Conversation`
- soft-deleted or purge-pending conversations must not accept new messages or delayed replies
- audit traces should be minimized so they do not become a shadow transcript store

### Architectural Implication
Deletion and purge are conversation-root and account-root concerns, not independent workflows for each derived table. The async worker must respect lifecycle state before executing delayed replies or background updates.

## Runtime Flows
### 1. Guest Trial Flow
1. Player opens the responsive web app.
2. System creates a guest session.
3. Player selects a character and starts chatting.
4. Guest Trial Gate counts only player input messages.
5. After 10 user input messages, further chat requires login.
6. If the guest logs in, guest state resets and does not carry over.

### 2. Authenticated Chat Flow
1. Player signs in with Facebook.
2. API resolves or creates a persistent user account.
3. Player selects a character.
4. Chat Orchestrator loads:
   - character profile
   - recent message context
   - long-term memory summary
   - relationship state
   - policy state
5. Policy module validates the request against age gate and unsafe-role rules.
6. Relationship Engine determines contextual tone and reply-delay tendency.
7. Chat Orchestrator schedules a reply job rather than always answering immediately.
8. Async worker executes the reply when the scheduled delay expires.
9. Provider Adapter calls the hosted LLM.
10. Output passes through post-generation safety checks.
11. Message is stored, displayed to the user, and memory plus scores are updated.

### 3. Returning Player Flow
1. Player logs in with Facebook.
2. System loads prior conversations by user-character pair.
3. Reopening a conversation restores recent context, long-term memory, and relationship state.
4. No other character receives that memory.

### 4. Low-Interest Delayed Reply Flow
1. Relationship Engine computes low `interest / reply priority` or high conflict.
2. Delay scheduler chooses a later delivery time, capped at 5 minutes.
3. Client shows simple delayed-message behavior rather than a complex activity simulation.

## Prompt Assembly Architecture
The prompt for each response should be assembled from distinct layers rather than a single unstructured transcript.

### Recommended Prompt Layers
1. System policy layer
   - unsafe-role restrictions
   - illegal action restrictions
   - age-gated sexual-content policy
2. Character layer
   - persona
   - age and background
   - interests and style
   - knowledge boundaries
3. Relationship layer
   - current visible and secret state metrics
   - current tone guidance
4. Memory layer
   - long-term summary
   - salient remembered facts
5. Recent conversation layer
   - recent messages only
6. Current user input

This layered structure keeps the provider abstraction clean and limits prompt drift.

## Relationship Engine Design
### Recommendation
Use deterministic score updates as the authoritative source, optionally fed by structured labels inferred from the latest conversation turn.

### Why
- easier tuning
- easier debugging
- lower risk of unstable progression
- safer premium score exposure in later phases

### MVP Processing Model
For each completed exchange, the engine should:
1. inspect the latest user and character messages
2. classify relevant interaction signals
3. apply bounded score deltas
4. recompute behavior thresholds
5. persist the updated state

### Output Consumers
The Relationship Engine should influence:
- tone selection
- topic depth
- flirt readiness
- delayed-reply tendency
- refusal intensity when boundaries are pressured

## Memory Architecture
### Short-Term Memory
Purpose:
Maintain coherence for the active conversation.

Recommended shape:
- recent raw messages
- rolling condensed session context

### Long-Term Memory
Purpose:
Store durable relationship and user facts for the specific user-character pair.

Recommended shape:
- stable user facts
- repeated preferences
- relationship milestones
- summarized prior emotional context

### Memory Update Strategy
Recommendation:
Update memory asynchronously after each completed exchange or short batch of exchanges.

Reason:
This reduces synchronous latency and allows better memory compaction.

## Safety And Policy Architecture
### Hard Guardrails
Every reply must respect:
- no important life-decision making for the player
- no illegal encouragement
- no serious real-world problem-solver role

### Age-Gated Content Controls
Phase 1 policy gate:
- simple self-attested 18+ confirmation dialog
- under-18 users do not receive sexual content
- adult-content behavior remains subject to hosted-provider feasibility

### Policy Enforcement Pipeline
1. pre-generation request policy check
2. prompt-layer policy injection
3. post-generation safety validation
4. fallback rewrite or refusal if needed

### Policy Mode Recommendation
Implement a policy capability flag with at least two modes:
- `hosted-safe-mode`: non-explicit romance/friendship only
- `adult-enabled-mode`: future mode only if provider and product policy permit it

This avoids architectural churn if hosted-provider policy blocks the intended adult-content behavior.

## Identity And Session Model
### Phase 1
- guest session for anonymous trial
- Facebook login for persistent identity
- guest state resets on login

### Future Phase
- linked identities for Facebook, LINE, email

### Architectural Implication
Identity must be abstracted behind an internal account model rather than embedding Facebook IDs everywhere. This allows future multi-provider linking without refactoring conversation ownership.

## Deployment Topology
### Phase 1 Recommendation
Deploy as a small number of containers on one medium-sized instance:
- web client container
- backend API container
- async worker container
- relational database container or managed equivalent
- in-memory cache/queue container or managed equivalent

### Why This Fits The MVP
- aligns with cost and operational simplicity goals
- supports delayed jobs without microservices
- keeps the architecture modular enough for later split-out

## Scalability Path
When load grows, the recommended split order is:
1. separate async worker from API
2. externalize cache/queue to managed infrastructure
3. isolate provider adapter and memory processing if needed
4. move to local LLM serving in phase 2

The internal module boundaries should mirror this future split even while everything runs in one codebase initially.

## Observability And Auditability
The system should capture:
- request and response timings
- scheduled versus actual reply delivery times
- provider latency and error rates
- safety refusals and fallback rewrites
- score changes per exchange
- memory update events
- guest-trial conversion to login

This is necessary because the product’s quality depends on conversational feel, not just uptime.

## Architecture Risks
### 1. Hosted Provider Policy Mismatch
Risk:
The hosted public model may not support the intended adult-content behavior even for self-attested 18+ users.

Mitigation:
Implement policy capability flags so phase 1 can degrade to non-explicit romance without redesign.

### 2. Score System Feels Arbitrary
Risk:
Relationship progression may feel fake if score updates are too opaque.

Mitigation:
Keep the scoring engine rule-driven and bounded.

### 3. Memory Bloat
Risk:
Long-term memory text may grow too large for prompt efficiency.

Mitigation:
Use asynchronous summarization and store only salient, compact memories.

### 4. Delay Behavior Feels Artificial
Risk:
Reply timing can feel fake if delay logic is random rather than state-driven.

Mitigation:
Tie delay behavior to explicit relationship and interest signals.

### 5. Guest Trial Abuse
Risk:
Anonymous users may repeatedly restart guest trials.

Mitigation:
Use browser session controls, device/browser heuristics, and rate limits, while accepting that hard prevention is not the MVP goal.

## Implementation Roadmap
### Stage 1: Foundation
- responsive web client shell
- backend API skeleton
- guest session and Facebook auth
- character catalog for Aira, Lyra, and Nova

### Stage 2: Core Chat Loop
- authenticated and guest chat
- provider adapter for hosted LLM
- delayed reply scheduling
- basic persistence for messages and conversations

### Stage 3: Memory And Scoring
- short-term context assembly
- long-term memory summarization
- relationship engine with deterministic updates

### Stage 4: Safety And Hardening
- policy enforcement pipeline
- age-gate handling
- observability and audit logging
- performance and cost tuning

## Open Questions
1. Can the phase 1 hosted model and provider policy support the intended adult-content policy for players who confirm they are 18+, or must adult sexual content wait for a later phase or local model?
2. What concrete retention durations should be assigned to guest-limited, account-durable, and audit-minimum data classes?
3. Should phase 1 expose any user-facing delete capability, or should delete remain an internal or admin-managed lifecycle path until later?

## Recommendation Summary
Recommendation:
Adopt the modular monolith plus async worker architecture in phase 1, keep memory strictly scoped per user-character pair, preserve an internal provider abstraction, and implement a policy capability switch so hosted-model limitations do not force a system redesign later.