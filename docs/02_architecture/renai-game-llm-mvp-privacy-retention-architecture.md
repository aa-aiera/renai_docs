# Renai Game LLM MVP - Privacy, Retention, And Data Lifecycle Architecture

## Document Control
- Status: Draft Architecture Companion
- Version: v1.1.0
- Last Updated: 2026-04-26
- Owner: SA

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.1.0 | Major | Reconciled lifecycle design to PRD v3.0.2 by removing guest lifecycle assumptions and defining explicit phase 1 retention windows plus admin-managed deletion behavior. | Persistence design, API planning, worker purge logic, and audit storage must align to the fixed retention windows and internal delete workflow. |
| 2026-04-26 | v1.0.0 | Major | Established the privacy, retention, and lifecycle companion as a managed architecture artifact. | Technical Lead planning and implementation tracking should preserve these lifecycle boundaries as part of Architecture v1.0.0. |

## Upstream Baseline
- Based On: Phase 1 MVP PRD v3.0.2 and MVP architecture packet v1.1.0

## Executive Summary
This document defines the phase 1 privacy, retention, and deletion architecture for the renai-game-style LLM product. The current PRD baseline now fixes the lifecycle values that were previously open: conversation data is retained for 12 months after last activity, derived memory and relationship state are hard-deleted within 30 days after confirmed deletion, audit-minimum traces are retained for 90 days, and phase 1 deletion is internal or admin-managed rather than Player self-service.

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-security-architecture.md`

## Lifecycle Goals
- Preserve Player-MC conversation continuity without indefinite retention.
- Ensure derived memory and relationship state never outlive the owning conversation after deletion.
- Keep audit traces minimal and separate from Player-visible transcript storage.
- Support admin-managed deletion without exposing a Player-facing delete control in phase 1.

## Data Classes And Retention Windows
| Data Class | Included Data | Retention Window | Delete Behavior |
| --- | --- | --- | --- |
| Conversation history | conversation records, message transcript, restore metadata | 12 months after last conversation activity unless earlier deletion is triggered | inaccessible immediately after delete intent; hard-deleted by purge workflow |
| Derived memory and relationship state | short-term context, long-term memory, relationship state, reply schedules | owner-bound during normal use; hard-delete within 30 days after confirmed deletion | cascades from owning conversation |
| Audit-minimum traces | metadata-first safety, operational, and abuse-review events | 90 days | retained separately from transcript store; purged on audit schedule |

## Ownership Model
### Primary Ownership Root
`Conversation` is the lifecycle root for:
- `Message`
- `ShortTermContext`
- `LongTermMemory`
- `RelationshipState`
- `ReplySchedule`

### Separate Retention Domain
`AuditTrace` is not a child retention root for Player-visible chat. It has its own 90-day retention policy and stricter access expectations.

## Lifecycle States
### Conversation States
- `active`: normal read and write access
- `soft-deleted`: Player-facing access blocked and new writes denied
- `purge-pending`: queued for irreversible cleanup
- `purged`: removed from the product data path

### State Semantics
- `soft-deleted` exists to stop access immediately while allowing asynchronous purge work
- `purge-pending` exists to coordinate worker-safe cleanup across owner-bound records
- delayed reply jobs must treat both `soft-deleted` and `purge-pending` as non-runnable states

## Deletion Policy
### Phase 1 Delete Exposure
Phase 1 does not expose a Player-facing self-service delete feature.

Deletion is initiated through an internal or admin-managed workflow that:
1. records delete intent
2. marks the conversation or account inaccessible
3. schedules purge work
4. completes hard deletion inside the approved purge window

### Conversation Delete Workflow
1. Internal/admin request marks the conversation `soft-deleted`.
2. API read and write paths stop returning it in normal Player flows.
3. Worker jobs stop executing delayed replies for that scope.
4. Purge orchestration marks the conversation `purge-pending`.
5. Owner-bound transcript and derived artifacts are hard-deleted within 30 days.

### Account Delete Workflow
1. Internal/admin account delete intent targets the Player account.
2. All owned conversations are marked for delete processing.
3. Owner-bound transcript and derived artifacts follow the same purge rule.
4. Audit traces remain on their separate 90-day retention rule unless a stricter legal or operational rule applies.

## Retention Enforcement Rules
### Conversation History
- Calculate retention from `last_activity_at`.
- Do not keep transcript data indefinitely for dormant conversations.
- If a conversation is deleted earlier, deletion overrides the 12-month durability window.

### Derived Memory And Relationship State
- Never treat derived memory as an independent retention root.
- Purge within 30 days after the owning conversation or account is confirmed for deletion.
- Ensure background jobs cannot recreate purged state after deletion.

### Audit-Minimum Traces
- Store only the minimum metadata and evidence needed for abuse review, policy enforcement, and incident investigation.
- Default to metadata-first payloads and reason codes rather than full transcript duplication.
- Keep audit access paths separate from Player-facing product reads.

## Operational Controls
### API Layer
- Reject reads and writes for `soft-deleted` and `purge-pending` conversations.
- Do not expose internal delete mechanics as public Player API surface in phase 1.

### Worker Layer
- Recheck lifecycle state before delayed reply generation.
- Cancel or skip jobs for non-active conversations.
- Run purge jobs idempotently.

### Storage Layer
- Preserve explicit lifecycle columns and purge timestamps.
- Support cascading cleanup from conversation ownership.
- Keep audit traces in separable storage or logical tables.

## Risks And Mitigations
### Risk 1: Over-Retention Of Derived Data
If memory summaries or relationship state survive after transcript deletion, behaviorally sensitive information remains.

Mitigation:
Keep derived state owner-bound and purge it within 30 days of confirmed deletion.

### Risk 2: Audit Store Becomes A Shadow Transcript
If audit traces store too much raw text, the system defeats its own retention minimization goals.

Mitigation:
Keep audit traces metadata-first and independently retained.

### Risk 3: Async Jobs Reanimate Deleted State
Delayed workers may create new messages or derived memory after delete intent.

Mitigation:
Require lifecycle rechecks before any async execution and make purge jobs idempotent.

## Non-Blocking Future Considerations
- A later phase may add Player-facing delete or export controls.
- A later phase may refine audit payload rules if moderation needs expand.
- A later phase may revise retention windows if legal or business requirements change.

## Recommendation Summary
Recommendation:
Treat the conversation as the only Player-visible retention root, retain transcript continuity for 12 months after last activity, purge derived memory and relationship state within 30 days after confirmed deletion, keep audit traces on a separate 90-day metadata-first policy, and keep delete initiation internal or admin-managed for the phase 1 MVP.
