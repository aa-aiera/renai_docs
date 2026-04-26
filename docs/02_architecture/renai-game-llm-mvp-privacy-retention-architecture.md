# Renai Game LLM MVP - Privacy, Retention, And Data Lifecycle Architecture

## Document Status
- Status: Draft Architecture Companion
- Date: 2026-04-26
- Based On: Phase 1 MVP PRD and MVP architecture packet

## Executive Summary
This document defines the phase 1 privacy, retention, and deletion architecture for the renai-game-style LLM product. The existing architecture packet already defines identity, conversation, memory, scheduling, and provider boundaries, but it leaves lifecycle rules under-specified. This companion closes that gap by defining how data should be classified, retained, expired, soft-deleted, hard-deleted, and kept auditable.

Because the source notes do not define final legal or business retention windows, this document does not invent exact retention durations beyond what the MVP already implies, such as guest-session expiry behavior. Instead, it defines lifecycle classes, ownership rules, deletion states, and implementation boundaries that allow final policy values to be filled in later without redesign.

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-erd.md`
- `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md`
- `docs/02_architecture/renai-game-llm-mvp-component-diagrams.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`

## Problem Statement
The MVP stores sensitive conversational data, age-confirmation state, relationship metrics, memory summaries, and authentication-linked identity. The current packet describes where that data lives, but not enough about how long it should live, what happens at guest login reset, how delete intent should propagate across derived data, or which records may need limited audit retention.

Without explicit lifecycle architecture, the implementation risks:
- keeping data longer than necessary
- deleting user-visible data while leaving derived memory behind
- weakening guest-reset guarantees
- making future privacy policy or deletion features expensive to add

## Scope
This document covers:
- data classification for phase 1 entities
- guest and authenticated lifecycle boundaries
- retention classes instead of final legal durations
- deletion-state architecture
- derived-data cleanup rules
- auditability boundaries

This document does not cover:
- jurisdiction-specific legal advice
- final product copy for privacy notices
- encryption or secrets-management implementation details
- multi-region data residency strategy

## Architecture Goals
- preserve the approved guest-reset rule
- make deletion propagate consistently across raw and derived data
- minimize retained personal and behavioral data where possible
- keep the schema and modules ready for future policy hardening
- avoid breaking observability and abuse-tracing requirements

## Data Classification Model
### Class A: Identity And Access Data
Includes:
- `UserAccount`
- `IdentityLink`
- guest-session identity markers
- session and age-confirmation state

Sensitivity:
High

Lifecycle intent:
- durable for authenticated accounts while the account remains active
- short-lived for guest usage
- deletion must remove or irreversibly disassociate downstream personal ownership where product policy allows

### Class B: Conversation Content Data
Includes:
- `Conversation`
- `Message`
- `ShortTermContext`

Sensitivity:
High

Lifecycle intent:
- durable for authenticated continuity in phase 1
- guest-scoped content expires with guest lifecycle and does not transfer on login
- delete requests must remove both raw content and prompt-ready condensed forms

### Class C: Derived Relationship And Memory Data
Includes:
- `LongTermMemory`
- `RelationshipState`
- prompt-ready summaries or salience records

Sensitivity:
High

Lifecycle intent:
- derived data must never outlive its owning conversation in normal operation
- deletion of a conversation or account must cascade into derived memory and relationship state

### Class D: Scheduling And Operational Data
Includes:
- `ReplySchedule`
- rate-limit counters
- transient queue payloads

Sensitivity:
Medium

Lifecycle intent:
- retained only while operationally necessary
- expired or completed records should age out faster than user-visible conversation history unless audit value requires otherwise

### Class E: Audit And Safety Trace Data
Includes:
- `AuditEvent`
- policy refusal events
- provider error traces with minimized conversational payload inclusion

Sensitivity:
Medium to High depending on payload

Lifecycle intent:
- should retain enough traceability for safety review and debugging
- should avoid storing more raw user text than necessary
- should remain separable from main user-facing chat history for controlled retention and access rules

## Lifecycle Architecture
### 1. Guest Session Lifecycle
Guest usage is intentionally disposable.

Rules:
- guest sessions are created at first app bootstrap
- guest conversations and messages are owned by `GuestSession`, not `UserAccount`
- guest memory and relationship state are allowed only within that guest-owned conversation scope
- on login, guest state is not migrated
- guest session data should move to an expired state and then be eligible for cleanup

Architectural implication:
Guest-reset behavior must be enforced in both API logic and persistence ownership. No derived memory artifact should survive as authenticated continuity after login.

### 2. Authenticated Account Lifecycle
Authenticated conversations are durable by default in phase 1.

Rules:
- authenticated users reopen the same user-character conversation root unless reset or archival behavior is added later
- memory and relationship state remain attached to that durable conversation root
- account deactivation or delete intent must propagate to owned conversation content and derived state

### 3. Conversation Lifecycle States
Recommendation:
Support lifecycle states conceptually even if the phase 1 UI exposes only active conversations.

Recommended states:
- `active`
- `inactive`
- `soft-deleted`
- `purge-pending`
- `purged`

Purpose:
These states allow operational safety, delayed background cleanup, and deletion-request processing without immediate irreversible data loss on the synchronous user path.

### 4. Derived Data Lifecycle Rule
`ShortTermContext`, `LongTermMemory`, and `RelationshipState` are derivative or conversation-bound records.

Rule:
They must follow the lifecycle of the owning `Conversation` and must never become independent retention roots.

## Retention Class Model
Because exact durations are not yet approved, the architecture should use retention classes rather than hardcoded day counts.

### Retention Class R1: Ephemeral
Examples:
- transient queue payloads
- cache entries
- short-lived rate-limit counters

Policy intent:
delete automatically after operational usefulness ends

### Retention Class R2: Guest-Limited
Examples:
- `GuestSession`
- guest-owned conversations
- guest-owned messages
- guest-owned memory artifacts

Policy intent:
retain only for the guest session lifecycle and cleanup grace period, then purge

### Retention Class R3: Account-Durable
Examples:
- authenticated conversations
- authenticated message history
- authenticated long-term memory
- relationship state

Policy intent:
retain while the account and conversation remain active, subject to future user deletion or archival policy

### Retention Class R4: Audit-Minimum
Examples:
- selected `AuditEvent` records
- safety and moderation traces

Policy intent:
retain only the minimum needed for debugging, safety review, abuse investigation, and compliance support

## Deletion Architecture
### Deletion Principles
- user-visible delete intent must propagate to raw and derived data
- guest reset is not the same as authenticated delete; it is a lifecycle boundary
- operational cleanup should be asynchronous once a delete or purge state is recorded
- audit retention should be minimized and decoupled from the main chat history

### Recommended Deletion States
For account-owned and conversation-owned data, use these logical stages:
1. `active`
2. `soft-deleted`
3. `purge-pending`
4. `purged`

### Soft Delete Purpose
Soft delete exists to:
- prevent immediate user-facing access
- block new writes to the deleted ownership scope
- allow background cascades to complete safely

### Purge-Pending Purpose
Purge-pending exists to:
- queue cleanup jobs for messages, memory, relationship state, schedules, and derived summaries
- cancel not-yet-executed delayed replies
- avoid partial deletion in synchronous request handling

### Hard Purge Purpose
Hard purge exists to:
- remove raw conversation text
- remove derived memory and relationship data
- remove active scheduling records
- remove or anonymize operational traces according to audit-minimum policy

## Entity-Level Lifecycle Guidance
| Entity | Retention Class | Delete Behavior | Notes |
| --- | --- | --- | --- |
| `GuestSession` | R2 | expire then purge | guest session must never become authenticated continuity |
| `Conversation` guest-owned | R2 | expire then purge | do not transfer on login |
| `Conversation` authenticated | R3 | soft-delete then purge | future UI may expose delete or archive later |
| `Message` guest-owned | R2 | cascade from guest conversation | raw guest text should not survive beyond guest retention |
| `Message` authenticated | R3 | cascade from authenticated conversation | player-visible history root |
| `ShortTermContext` | follows owner | cascade from conversation | condensed context must not outlive source conversation |
| `LongTermMemory` | follows owner | cascade from conversation | derived summary must be deleted with source scope |
| `RelationshipState` | follows owner | cascade from conversation | structured derived state cannot survive deleted conversation |
| `ReplySchedule` | R1 or owner-bound | cancel and purge | do not execute for deleted or expired conversations |
| `AuditEvent` | R4 | minimize and separate | prefer event metadata over raw transcript copies |
| `IdentityLink` | R3 | delete with account or unlink policy | exact unlink workflow deferred |

## Operational Flows
### Flow 1: Guest Login Reset
1. Guest logs in with Facebook.
2. Authenticated `UserAccount` is resolved.
3. Guest-owned conversations remain guest-owned and inaccessible to the new authenticated session.
4. Guest ownership moves toward expiry and purge.
5. No guest-derived `LongTermMemory` or `RelationshipState` is copied into the authenticated account.

### Flow 2: Authenticated Conversation Delete Intent
1. User or admin delete intent marks the conversation `soft-deleted`.
2. Read APIs stop returning it.
3. New writes and delayed replies are blocked.
4. Background purge marks cleanup tasks for messages, short-term context, long-term memory, relationship state, and reply schedules.
5. Audit traces are minimized or anonymized according to audit-minimum policy.

### Flow 3: Account Delete Intent
1. Account is marked deletion-requested or equivalent lifecycle state.
2. All owned authenticated conversations move to `purge-pending` through a controlled cascade.
3. Identity links are removed or disassociated.
4. Remaining audit traces are minimized under separate retention rules.

## Privacy By Design Requirements
### Data Minimization
- do not store provider-specific raw request and response envelopes unless clearly needed for debugging
- keep `AuditEvent` payloads metadata-focused where possible
- avoid duplicating full raw transcripts into multiple tables

### Ownership Integrity
- every memory and relationship artifact must point back to one owning conversation
- no orphaned memory summaries may exist after conversation purge

### Access Separation
- guest-owned data and authenticated-owned data must remain distinct access domains
- admin or moderation access, if added later, should use audit-specific access paths rather than direct product-user flows

### Deletion Safety
- scheduled reply jobs must verify ownership state before execution
- purge workers must be idempotent so retries do not recreate or partially restore deleted state

## Schema And Module Impact
### Recommended Schema Extensions
If not already present in implementation design, the architecture should support:
- conversation lifecycle status values beyond only active state
- optional deletion-request timestamps for account and conversation roots
- purge execution timestamps where lifecycle traceability matters
- audit payload design that separates metadata from raw text content

### Module Ownership Impact
- `Identity Module` owns guest expiry and authenticated account lifecycle entrypoints
- `Guest Trial Gate` must respect expired guest state
- `Conversations Module` owns lifecycle status enforcement on read and write paths
- `Memory Module` and `Relationship Engine` must cascade cleanup from conversation state
- `Async Worker` must cancel or skip scheduled replies for deleted scopes
- `Audit Module` must enforce minimized trace payload design

## Risks And Trade-Offs
### Risk 1: Over-Retention Of Derived Data
If memory summaries and relationship state are treated as independent assets, deleted conversations may still persist behaviorally sensitive information.

Mitigation:
Make derived records owner-bound and cascade on deletion.

### Risk 2: Audit Payloads Become Shadow Chat History
If audit events store too much raw conversation text, audit logs effectively become a second transcript store.

Mitigation:
Prefer metadata, reason codes, and narrow excerpts only where operationally required.

### Risk 3: Guest Reset Becomes Inconsistent
If guest reset is enforced only in UI or session logic, downstream data may survive incorrectly.

Mitigation:
Keep guest ownership separate at the schema level and purge from ownership roots.

## Assumptions
- Final legal and business retention day counts are not yet approved.
- Phase 1 may not expose user-facing delete controls, but the architecture should still support deletion-state handling.
- Audit trace retention may need narrower access and separate review paths later.

## Implementation Roadmap Impact
### Immediate Phase 1 Requirements
- implement guest expiry and purge behavior
- make conversation lifecycle status explicit in the persistence model
- ensure delayed reply execution checks conversation lifecycle state before delivery
- ensure memory and relationship cleanup cascade from conversation ownership

### Near-Term Follow-Up
- define concrete retention windows per retention class
- define user-facing delete and export behavior if added post-MVP
- define admin and moderation access model for audit traces

## Open Questions
1. What concrete retention durations should be assigned to R2 guest-limited, R3 account-durable, and R4 audit-minimum classes?
2. Should phase 1 expose any user-facing delete capability, or should delete only exist as an internal/admin lifecycle path until later?
3. Should audit traces store narrow text excerpts at all, or should phase 1 restrict them to metadata and reason codes only?

## Recommendation Summary
Recommendation:
Adopt a lifecycle architecture where guest data is intentionally disposable, authenticated conversation continuity is durable but owner-bound, and all derived memory and relationship data cascades from the owning conversation. Use retention classes instead of hardcoded durations until policy values are approved, and prevent audit logs from becoming a shadow copy of user chat history.