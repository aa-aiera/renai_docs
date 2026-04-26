# Renai Game LLM MVP - Sequence Flows

## Document Status
- Status: Draft Sequence Flow Companion
- Date: 2026-04-26
- Based On: Phase 1 MVP PRD, MVP HLD, and MVP ERD

## Executive Summary
This document captures the main runtime sequence flows for the phase 1 MVP system. It complements the HLD and ERD by showing how the browser client, backend modules, storage layers, scheduler, and LLM provider interact during the most important product journeys.

The focus is on the approved MVP behavior:
- guest trial before login
- Facebook-based returning identity
- one-on-one chat only
- delayed human-like replies
- conversation-scoped memory
- hidden phase-1 relationship scores
- guest reset on login

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-erd.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`

## Scope
This document covers:
- guest trial start and message cap enforcement
- guest-to-login reset behavior
- authenticated chat request and delayed reply generation
- returning-player conversation restore
- post-reply memory and relationship update processing

This document does not cover:
- premium score visibility flows
- cross-character memory flows
- native mobile app flows
- multi-provider account linking flows beyond Facebook in phase 1

## Participants
| Participant | Role |
| --- | --- |
| Browser Client | Responsive browser app used by player |
| API / BFF | Client-facing backend entrypoint |
| Identity Module | Guest and Facebook identity handling |
| Guest Trial Gate | Enforces pre-login chat cap |
| Character Catalog | Loads character profile and metadata |
| Chat Orchestrator | Coordinates chat request processing |
| Policy Module | Enforces age gate and safety restrictions |
| Relationship Engine | Loads and updates relationship state |
| Memory Module | Loads and updates short-term and long-term memory |
| Provider Adapter | Normalized access to hosted public LLM |
| Async Worker | Executes delayed reply and background jobs |
| Relational Store | Persistent relational database |
| Queue / Cache | Delayed jobs and transient counters |

## Flow 1: Guest Trial Start And First Chat
### Purpose
Shows how an anonymous player starts using the product and sends an initial message before login.

```mermaid
sequenceDiagram
    autonumber
    participant C as Browser Client
    participant API as API / BFF
    participant ID as Identity Module
    participant GT as Guest Trial Gate
    participant CHAR as Character Catalog
    participant DB as Relational Store

    C->>API: Open app
    API->>ID: Create guest session
    ID->>DB: Insert GuestSession
    ID-->>API: guest_session_id
    API-->>C: App bootstrap + guest session

    C->>API: Browse characters
    API->>CHAR: Load character list
    CHAR->>DB: Read CharacterProfile rows
    CHAR-->>API: character list
    API-->>C: Render character list / swipe cards

    C->>API: Start chat with selected character
    API->>GT: Check guest input quota
    GT->>DB: Read GuestSession.guest_input_message_count
    GT-->>API: Allowed
    API->>DB: Create guest-owned Conversation if missing
    API-->>C: Chat screen ready
```

## Flow 2: Guest Message Send And Trial Limit Enforcement
### Purpose
Shows how the 10-input guest cap is enforced and how the user is blocked from continuing without login.

```mermaid
sequenceDiagram
    autonumber
    participant C as Browser Client
    participant API as API / BFF
    participant GT as Guest Trial Gate
    participant ORCH as Chat Orchestrator
    participant DB as Relational Store
    participant Q as Queue / Cache

    C->>API: Send guest user message
    API->>GT: Validate guest quota
    GT->>DB: Read guest_input_message_count

    alt count < 10
        GT->>DB: Increment guest_input_message_count
        GT-->>API: Allowed
        API->>ORCH: Accept message
        ORCH->>DB: Insert user Message
        ORCH->>Q: Enqueue delayed reply job
        ORCH-->>API: Accepted
        API-->>C: Message accepted, waiting for delayed reply
    else count >= 10
        GT-->>API: Blocked - login required
        API-->>C: Show login required state
    end
```

## Flow 3: Guest Login Reset
### Purpose
Shows the approved product rule that guest chat state does not migrate into the authenticated account after Facebook login.

```mermaid
sequenceDiagram
    autonumber
    participant C as Browser Client
    participant API as API / BFF
    participant ID as Identity Module
    participant DB as Relational Store

    C->>API: Continue with Facebook login
    API->>ID: Exchange auth result / verify identity
    ID->>DB: Upsert UserAccount
    ID->>DB: Upsert IdentityLink(Facebook)
    ID-->>API: authenticated user_account_id

    API->>DB: Mark guest session inactive or expired
    Note over API,DB: Guest conversations, memory, and relationship state are not migrated
    API-->>C: Start authenticated session with empty authenticated chat state
```

## Flow 4: Authenticated Chat Request And Delayed Reply Scheduling
### Purpose
Shows how the system accepts an authenticated message, assembles the required context, applies policy, and schedules a delayed reply instead of immediately returning provider output.

```mermaid
sequenceDiagram
    autonumber
    participant C as Browser Client
    participant API as API / BFF
    participant ORCH as Chat Orchestrator
    participant CHAR as Character Catalog
    participant MEM as Memory Module
    participant REL as Relationship Engine
    participant POL as Policy Module
    participant DB as Relational Store
    participant Q as Queue / Cache

    C->>API: Send authenticated user message
    API->>ORCH: Handle chat request
    ORCH->>CHAR: Load CharacterProfile
    CHAR->>DB: Read CharacterProfile
    CHAR-->>ORCH: Character metadata

    ORCH->>MEM: Load short-term and long-term context
    MEM->>DB: Read Conversation, Message, ShortTermContext, LongTermMemory
    MEM-->>ORCH: Prompt-ready memory context

    ORCH->>REL: Load current RelationshipState
    REL->>DB: Read RelationshipState
    REL-->>ORCH: Current scores and tone hints

    ORCH->>POL: Validate request against safety and age gate
    POL-->>ORCH: Request allowed / constrained mode

    ORCH->>DB: Insert user Message
    ORCH->>REL: Compute reply delay tendency
    REL-->>ORCH: Delay recommendation
    ORCH->>DB: Insert ReplySchedule
    ORCH->>Q: Enqueue scheduled reply job
    ORCH-->>API: Accepted for delayed processing
    API-->>C: User message stored, waiting for reply
```

## Flow 5: Delayed Reply Execution Through Hosted LLM
### Purpose
Shows how the worker executes the scheduled reply, calls the provider, applies post-generation checks, and delivers the character message.

```mermaid
sequenceDiagram
    autonumber
    participant W as Async Worker
    participant Q as Queue / Cache
    participant ORCH as Chat Orchestrator
    participant MEM as Memory Module
    participant REL as Relationship Engine
    participant POL as Policy Module
    participant LLM as Provider Adapter
    participant DB as Relational Store

    W->>Q: Poll due reply job
    Q-->>W: ReplySchedule job
    W->>ORCH: Execute scheduled reply

    ORCH->>MEM: Reload latest prompt context
    MEM->>DB: Read recent messages and long-term memory
    MEM-->>ORCH: Prompt layers

    ORCH->>REL: Load latest relationship state
    REL->>DB: Read RelationshipState
    REL-->>ORCH: Tone and behavior hints

    ORCH->>POL: Apply pre-generation policy mode
    POL-->>ORCH: Hosted-safe-mode or constrained mode

    ORCH->>LLM: Generate character reply
    LLM-->>ORCH: Candidate reply

    ORCH->>POL: Run post-generation validation
    alt reply passes
        POL-->>ORCH: Approved
    else reply fails
        POL-->>ORCH: Refuse or rewrite
    end

    ORCH->>DB: Insert character Message
    ORCH->>DB: Update ReplySchedule status
    ORCH-->>W: Reply delivered
```

## Flow 6: Post-Reply Memory And Score Update
### Purpose
Shows how relationship state and memory are updated after a completed exchange without blocking the visible user experience.

```mermaid
sequenceDiagram
    autonumber
    participant W as Async Worker
    participant ORCH as Chat Orchestrator
    participant REL as Relationship Engine
    participant MEM as Memory Module
    participant DB as Relational Store

    W->>ORCH: Start post-reply background processing

    ORCH->>REL: Analyze recent exchange
    REL->>DB: Read latest user and character messages
    REL-->>ORCH: Score deltas and behavior thresholds
    ORCH->>DB: Update RelationshipState

    ORCH->>MEM: Update short-term context
    MEM->>DB: Update ShortTermContext

    ORCH->>MEM: Evaluate long-term memory writeback
    MEM->>DB: Insert or supersede LongTermMemory rows

    ORCH->>DB: Insert AuditEvent entries
    ORCH-->>W: Background update complete
```

## Flow 7: Returning Player Reopens Existing Character Chat
### Purpose
Shows how a returning Facebook-authenticated player gets continuity with the same character.

```mermaid
sequenceDiagram
    autonumber
    participant C as Browser Client
    participant API as API / BFF
    participant ID as Identity Module
    participant MEM as Memory Module
    participant REL as Relationship Engine
    participant DB as Relational Store

    C->>API: Open app with authenticated session
    API->>ID: Resolve UserAccount from session
    ID-->>API: user_account_id

    C->>API: Open existing character chat
    API->>DB: Load Conversation by user_account_id + character_profile_id
    API->>MEM: Load recent context and long-term memory
    MEM->>DB: Read Message, ShortTermContext, LongTermMemory
    API->>REL: Load relationship state
    REL->>DB: Read RelationshipState

    API-->>C: Render restored chat context and conversation continuity
```

## Sequence Design Notes
### Why Reply Scheduling Happens Before Provider Call
The user experience depends on controlled timing, and some replies must be delayed by minutes. If the provider call happened synchronously inside the client request, the system would be forced into either long client blocking or fake frontend-only delays.

### Why Memory And Relationship Updates Are Post-Reply Jobs
This keeps the visible chat loop faster and reduces prompt assembly latency on the critical path.

### Why Guest Reset Is Explicitly Modeled
The product requirement says guest history, memory, and relationship state reset at login. Modeling this as a first-class sequence avoids accidental migration logic later.

### Why Policy Is Checked Both Before And After Generation
Pre-generation policy shapes what the model is allowed to attempt. Post-generation policy catches unsafe output that still slips through. The combination is necessary because the product relies on non-deterministic LLM behavior.

### Why Lifecycle State Must Be Checked Before Reply Execution
Delayed replies are asynchronous. By the time a job becomes due, the owning guest session or conversation may already be expired, soft-deleted, or purge-pending. The worker must therefore recheck lifecycle state before provider generation or message delivery.

## Cross-Document Consistency Checks
This sequence document is consistent with the companion architecture artifacts in these ways:
- matches the modular monolith plus async worker design in the HLD
- uses the conversation-centric persistence model from the ERD
- preserves guest reset behavior from the PRD
- preserves per-user-per-character memory isolation from the PRD and HLD
- preserves conversation-root lifecycle ownership from the privacy and retention architecture
- keeps phase 1 scores hidden from players

## Open Questions
1. Can the phase 1 hosted model and provider policy support the intended adult-content policy for players who confirm they are 18+, or must adult sexual content wait for a later phase or local model?

## Recommendation Summary
Recommendation:
Use these sequences as the reference runtime contract for implementation planning. The next architecture-to-engineering handoff should turn these flows into API contracts, job payload definitions, and module responsibilities in the implementation blueprint.