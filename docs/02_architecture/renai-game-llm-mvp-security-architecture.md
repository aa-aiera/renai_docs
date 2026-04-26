# Renai Game LLM MVP - Security Architecture

## Document Control
- Status: Draft Architecture Companion
- Version: v1.1.0
- Last Updated: 2026-04-26
- Owner: SA

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.1.0 | Major | Reconciled the security model to PRD v3.0.2 by removing guest-session security assumptions, aligning to login-required chat, and enforcing the hosted-model non-explicit plus admin-managed deletion baseline. | Planning and implementation work must remove guest-surface controls and focus on authenticated session security, lifecycle-safe workers, and metadata-first audit handling. |
| 2026-04-26 | v1.0.0 | Major | Established the security companion as a managed architecture artifact. | Technical Lead planning and implementation tracking should continue to treat these controls as part of the Architecture v1.0.0 baseline. |

## Upstream Baseline
- Based On: Phase 1 MVP PRD v3.0.2 and MVP architecture packet v1.1.0

## Executive Summary
This document defines the phase 1 security architecture for the renai-game-style LLM MVP. The security model now centers on authenticated Player sessions, strict Player-MC conversation ownership, hosted-model non-explicit policy enforcement, lifecycle-safe async jobs, server-side provider isolation, and metadata-first audit review.

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`

## Security Goals
- Ensure only the owning authenticated Player can access a conversation.
- Prevent direct client access to the hosted provider and provider credentials.
- Prevent async replies and derived-state updates from acting on deleted scopes.
- Keep audit traces operationally useful without duplicating transcript storage.
- Enforce the approved non-explicit hosted-model content posture.

## Trust Boundaries
| Boundary | Security Expectation |
| --- | --- |
| Browser to Backend API | authenticated session required for chat operations; no trust in client-owned ownership claims |
| Backend API to Database | server-side ownership and lifecycle checks on every sensitive operation |
| Backend API and Worker to Provider | provider credentials remain server-only; requests flow through the provider adapter |
| Worker to Queue / Cache | only approved job payloads are executable; lifecycle recheck required before side effects |
| Internal/Admin Operations to Product Data | privileged access is separate from Player-facing flows and should be auditable |

## Authentication And Session Security
### Player Identity
- Facebook is the only active phase 1 identity provider.
- Chat creation, message send, and conversation restore require an authenticated Player session.
- Session validation must happen server-side for every chat read or write.

### Session Requirements
- Session tokens or cookies must be integrity-protected and server-validated.
- Session expiration and invalidation must be enforced consistently in API and worker lookups.
- Player identity must not be inferred from client-submitted conversation identifiers alone.

## Authorization Model
### Conversation Access Rule
Every conversation access path must verify:
1. the requester is authenticated
2. the requester resolves to the owning `player_account_id`
3. the conversation is still in an accessible lifecycle state

### Lifecycle-Safe Authorization
- `soft-deleted` and `purge-pending` conversations are not readable or writable in normal Player flows
- worker execution must treat those states as hard stops

## Hosted-Model Content Security
### Approved Posture
Phase 1 uses a hosted public model in a non-explicit romance mode only.

### Controls
- policy checks run before generation and after generation
- disallowed explicit sexual content is blocked or rewritten server-side
- provider output is never trusted without post-generation validation
- no phase 1 endpoint attempts to unlock explicit adult sexual content

## Data Protection Controls
### Sensitive Data Categories
- Player identity and provider link data
- conversation transcript and delayed reply state
- memory summaries and relationship metrics
- audit and abuse-review traces

### Protection Rules
- secrets remain server-side only
- transcript and derived memory access are scoped by conversation ownership
- audit traces are separated from Player-facing transcript storage
- retention and purge rules are enforced as security controls, not just privacy preferences

## Async And Worker Security
### Delayed Reply Execution
Before any worker generates or stores an MC reply, it must revalidate:
- authenticated ownership context still exists
- conversation is `active`
- schedule has not been cancelled

### Purge And Cleanup Jobs
- purge jobs must be idempotent
- purge jobs must not recreate derived state after deletion
- reply jobs and memory jobs must fail closed on missing or deleted ownership scopes

## Abuse Resistance
### Primary Risks
- session theft or misuse
- conversation enumeration
- replay or spam of message-send operations
- provider misuse through malformed prompt requests
- audit over-collection

### Required Controls
- rate limiting on auth and message endpoints
- ownership checks on every conversation endpoint
- server-side policy and provider validation
- metadata-first audit emission
- privileged access review for internal delete workflows

## Operational Access
### Internal Or Admin-Managed Delete
Phase 1 deletion is not Player self-service.

Operational delete tooling must:
- require privileged access
- record audit traces for delete intent and purge completion
- avoid exposing raw transcript data unnecessarily to operators

### Audit Review Access
Audit review should prefer reason codes, event metadata, and narrow evidence over broad transcript browsing.

## Key Threat Scenarios
### 1. Unauthorized Conversation Access
Risk:
A client or attacker tries to read or write another Player's conversation.

Control:
Require server-side Player-to-conversation ownership validation on every request.

### 2. Async Execution Against Deleted State
Risk:
A delayed reply arrives after a conversation has been deleted.

Control:
Recheck lifecycle state inside the worker before provider calls and persistence.

### 3. Hosted-Model Output Violates Product Posture
Risk:
The provider emits content outside the approved non-explicit phase 1 boundary.

Control:
Enforce pre-generation and post-generation policy checks through the policy-safety module.

### 4. Audit Over-Retention
Risk:
Audit traces become a shadow transcript store.

Control:
Keep audit payloads metadata-first and subject to the 90-day audit retention window.

## Recommendation Summary
Recommendation:
Keep the phase 1 security model simple but explicit: authenticated Player sessions for chat, strict conversation-root authorization, server-only provider access, lifecycle-aware async execution, metadata-first audit storage, and hosted-model non-explicit policy enforcement as a required server-side control.
