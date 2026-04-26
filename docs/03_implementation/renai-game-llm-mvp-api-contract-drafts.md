# Renai Game LLM MVP - API Contract Drafts

## Document Control
- Status: Draft API Contracts
- Version: v1.2.2
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.2.2 | Minor | Bound the contract draft to the exact browser and runtime scaffold by documenting cookie-session client behavior and the config keys that drive browser-to-API connectivity. | Frontend and backend scaffolding can implement HTTP transport and session handling directly from the contract baseline instead of leaving client transport behavior implicit. |
| 2026-04-26 | v1.2.1 | Minor | Recorded the chosen React, Go, and cookie-session implementation baseline in the contract assumptions and resolved the remaining M0 transport and tooling questions. | Browser auth handling, backend middleware, and worker expectations should now assume cookie-backed sessions and the selected Go service stack. |
| 2026-04-26 | v1.2.0 | Major | Recorded the selected phase 1 technology baseline and mapped the draft contracts to a React browser client, Go backend services, PostgreSQL persistence, Redis cache, and RabbitMQ job delivery. | The implementation blueprint, execution backlog, and task artifact must treat stack family selection as locked while leaving framework-level details open. |
| 2026-04-26 | v1.1.0 | Major | Reconciled the contract drafts to PRD v3.0.2 and architecture v1.1.0 by removing guest APIs, adopting Player and MC terminology, requiring authenticated chat, and adding internal lifecycle delete handling. | Client, backend, worker, and lifecycle implementation work should treat these contracts as the current planning baseline. |
| 2026-04-26 | v1.0.0 | Major | Established the API contract drafts as a managed implementation-planning artifact. | Implementor execution should treat these contracts as part of the Planning v1.0.0 baseline until the architecture or planning packet changes. |

## Upstream Baseline
- Based On: MVP PRD v3.0.2, phase 1 architecture packet v1.1.0, and MVP implementation blueprint v1.2.2

## Executive Summary
This document defines draft API contracts for the phase 1 MVP browser client, backend API, and async worker job payloads. The contracts remain transport-neutral at the schema level, but they now map to a React browser client, Go backend API and worker services, PostgreSQL persistence, Redis cache usage, and RabbitMQ-delivered background jobs.

These contracts are designed for:
- authenticated Player chat only
- Facebook-based Player identity
- one-on-one Player-to-MC conversation restore
- delayed MC replies
- conversation-scoped memory updates
- hidden internal relationship scores in phase 1
- internal or admin-managed lifecycle delete handling

## Source Notes
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-erd.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`
- `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md`
- `docs/03_implementation/renai-game-llm-mvp-implementation-blueprint.md`

## Conventions
### API Prefix
Recommendation:
Use `/v1/` for Player-facing MVP endpoints and `/internal/v1/` for privileged lifecycle operations.

### Content Type
- Request: `application/json`
- Response: `application/json`

### Concrete Runtime Mapping
- React browser client calls the Go backend over HTTP JSON.
- React is expected to use TypeScript, Vite, React Router, TanStack Query, and Zustand for the phase 1 browser shell.
- Go services are expected to expose these HTTP contracts through `chi` on top of `net/http`.
- Go backend services persist durable state in PostgreSQL.
- Redis supports cache and transient coordination concerns around session-adjacent and rate-limited flows.
- RabbitMQ delivers delayed reply, memory, relationship, audit, and purge jobs to the Go worker runtime.

### Session Transport Mapping
- The backend sets a secure, `HttpOnly`, `SameSite=Lax` session cookie containing an opaque session ID.
- The browser relies on cookie transport rather than bearer tokens in browser storage.
- Session state is server-side and Redis-backed.
- State-changing routes validate request origin in addition to cookie security settings.

### Client Transport Binding
- The React HTTP client must send requests with `credentials: include`.
- `VITE_API_BASE_URL` is the single browser-visible config source for API routing.
- Backend cookie and origin policy must derive from `PUBLIC_APP_ORIGIN`, `SESSION_COOKIE_NAME`, `SESSION_COOKIE_DOMAIN`, and `SESSION_COOKIE_SECURE`.

### Common Response Envelope
Recommendation:
Use a consistent envelope shape.

```json
{
  "data": {},
  "meta": {},
  "error": null
}
```

### Identity Semantics
- discovery endpoints may be public if the product chooses to expose browse-only entry without chat access
- chat endpoints require an authenticated Player session
- authenticated requests carry a resolved `player_account_id` context
- no guest or anonymous chat owner scope exists in phase 1

### Lifecycle And Retention Semantics
- conversations may move through lifecycle states such as `active`, `soft-deleted`, and `purge-pending`
- non-active conversations are not returned to normal Player chat flows and do not accept new writes or delayed replies
- derived memory and relationship state follow the owner conversation lifecycle and are not exposed as independent client resources
- public delete endpoints are not part of phase 1

## HTTP API Contracts
### 1. Inspect Current Session
`GET /v1/me/session`

Purpose:
Return the current browser session view and whether chat access is available.

Response:
```json
{
  "data": {
    "session_type": "authenticated-or-anonymous",
    "player_account_id": "uuid-or-null",
    "chat_access": {
      "login_required": true,
      "can_chat": true
    },
    "content_mode": "non_explicit_phase1"
  },
  "meta": {},
  "error": null
}
```

### 2. Facebook Auth Exchange
`POST /v1/auth/facebook/exchange`

Purpose:
Resolve Facebook login into an internal authenticated session.

Request:
```json
{
  "facebook_auth_token": "opaque-token"
}
```

Response:
```json
{
  "data": {
    "player_account_id": "uuid",
    "session_type": "authenticated",
    "chat_access": {
      "login_required": true,
      "can_chat": true
    }
  },
  "meta": {},
  "error": null
}
```

### 3. MC List
`GET /v1/mcs`

Purpose:
Load the MVP roster for discovery screens.

Response:
```json
{
  "data": [
    {
      "mc_id": "uuid",
      "code_name": "aira",
      "display_name": "Aira",
      "profile_summary": "Warm, witty, observant",
      "avatar_url": "string-or-null",
      "content_mode": "non_explicit_phase1"
    }
  ],
  "meta": {
    "count": 3
  },
  "error": null
}
```

### 4. MC Detail
`GET /v1/mcs/{mc_id}`

Purpose:
Load detailed MC profile information for discovery and chat context.

Response:
```json
{
  "data": {
    "mc_id": "uuid",
    "display_name": "Aira",
    "age_years": 22,
    "persona_summary": "Warm, witty, observant",
    "talking_style_summary": "Playful teasing, casual Thai tone",
    "interests_summary": "Art, music, campus life, coffee",
    "relationship_boundary_summary": "Romance requires trust and comfort",
    "content_mode": "non_explicit_phase1"
  },
  "meta": {},
  "error": null
}
```

### 5. Get Or Create Conversation For MC
`POST /v1/conversations/by-mc/{mc_id}`

Purpose:
Resolve the active conversation root for the authenticated Player and selected MC.

Response:
```json
{
  "data": {
    "conversation_id": "uuid",
    "mc_id": "uuid",
    "player_account_id": "uuid",
    "conversation_status": "active"
  },
  "meta": {},
  "error": null
}
```

### 6. Load Conversation State
`GET /v1/conversations/{conversation_id}`

Purpose:
Return the chat state needed to render the conversation view.

Response:
```json
{
  "data": {
    "conversation_id": "uuid",
    "conversation_status": "active",
    "mc": {
      "mc_id": "uuid",
      "display_name": "Aira"
    },
    "messages": [
      {
        "message_id": "uuid",
        "sender_type": "player",
        "message_text": "hello",
        "delivery_state": "delivered",
        "created_at": "2026-04-26T10:00:00Z"
      }
    ]
  },
  "meta": {},
  "error": null
}
```

### 7. Send Player Message
`POST /v1/conversations/{conversation_id}/messages`

Purpose:
Accept a Player message, persist it, and schedule a delayed MC reply.

Request:
```json
{
  "message_text": "How was your day?"
}
```

Successful Response:
```json
{
  "data": {
    "accepted": true,
    "message": {
      "message_id": "uuid",
      "sender_type": "player",
      "message_text": "How was your day?",
      "delivery_state": "accepted",
      "created_at": "2026-04-26T10:05:00Z"
    },
    "reply_schedule": {
      "reply_schedule_id": "uuid",
      "scheduled_for": "2026-04-26T10:05:03Z"
    }
  },
  "meta": {},
  "error": null
}
```

Auth Required Response:
```json
{
  "data": null,
  "meta": {
    "login_required": true
  },
  "error": {
    "code": "AUTH_REQUIRED",
    "message": "Login is required before chat is available."
  }
}
```

### 8. Poll Latest Conversation Messages
`GET /v1/conversations/{conversation_id}/messages?after={message_id}`

Purpose:
Allow the client to refresh newly delivered MC replies.

Response:
```json
{
  "data": [
    {
      "message_id": "uuid",
      "sender_type": "mc",
      "message_text": "I just got back from class.",
      "delivery_state": "delivered",
      "created_at": "2026-04-26T10:05:03Z",
      "delivered_at": "2026-04-26T10:05:03Z"
    }
  ],
  "meta": {},
  "error": null
}
```

### 9. List Player Conversations
`GET /v1/me/conversations`

Purpose:
Return the authenticated Player's conversation list for chat restore and resume.

Response:
```json
{
  "data": [
    {
      "conversation_id": "uuid",
      "mc_id": "uuid",
      "mc_display_name": "Aira",
      "last_activity_at": "2026-04-26T10:05:03Z",
      "last_message_preview": "I just got back from class."
    }
  ],
  "meta": {},
  "error": null
}
```

### 10. Internal Lifecycle Delete Request
`POST /internal/v1/lifecycle/conversations/{conversation_id}/delete`

Purpose:
Accept an internal or admin-managed delete request for a conversation and enqueue purge work.

Request:
```json
{
  "reason_code": "player_support_or_policy_request"
}
```

Response:
```json
{
  "data": {
    "conversation_id": "uuid",
    "conversation_status": "soft-deleted",
    "purge_enqueued": true
  },
  "meta": {},
  "error": null
}
```

## Worker Job Payload Drafts
### 1. Execute Delayed Reply Job
Job Name:
`execute_delayed_reply`

Payload:
```json
{
  "reply_schedule_id": "uuid",
  "conversation_id": "uuid",
  "player_account_id": "uuid",
  "mc_id": "uuid"
}
```

### 2. Update Short-Term Context Job
Job Name:
`update_short_term_context`

Payload:
```json
{
  "conversation_id": "uuid",
  "trigger_message_id": "uuid"
}
```

### 3. Update Long-Term Memory Job
Job Name:
`update_long_term_memory`

Payload:
```json
{
  "conversation_id": "uuid",
  "window_end_message_id": "uuid"
}
```

### 4. Update Relationship State Job
Job Name:
`update_relationship_state`

Payload:
```json
{
  "conversation_id": "uuid",
  "player_message_id": "uuid",
  "mc_message_id": "uuid"
}
```

### 5. Purge Conversation Job
Job Name:
`purge_conversation`

Payload:
```json
{
  "conversation_id": "uuid",
  "purge_after_at": "2026-05-26T10:00:00Z"
}
```

### 6. Emit Audit Event Job
Job Name:
`emit_audit_event`

Payload:
```json
{
  "conversation_id": "uuid",
  "event_type": "string",
  "actor_scope": "system-or-module",
  "event_payload": {}
}
```

## Error Contract Draft
Recommendation:
Use stable error codes for user-facing and system-facing failures.

| Code | Meaning |
| --- | --- |
| `AUTH_REQUIRED` | Chat operation requires an authenticated Player session |
| `SESSION_EXPIRED` | Authenticated session is expired or invalid |
| `MC_NOT_FOUND` | MC is missing or unavailable |
| `CONVERSATION_NOT_FOUND` | Conversation is missing or does not belong to caller |
| `CONVERSATION_UNAVAILABLE` | Conversation exists but is deleted, purge-pending, or otherwise inaccessible |
| `POLICY_BLOCKED` | Request or output blocked by hard safety rules |
| `RATE_LIMITED` | Request was throttled by abuse-resistance controls |
| `PROVIDER_UNAVAILABLE` | Hosted LLM call failed or timed out |
| `REPLY_NOT_READY` | Delayed reply exists but is not yet delivered |
| `ADMIN_AUTH_REQUIRED` | Internal lifecycle endpoint requires privileged access |

## Contract Notes
### Phase 1 Score Visibility
The API should not expose relationship scores to Players in phase 1.

### Phase 1 Content Posture
The contracts assume a hosted public model in non-explicit mode only. No age-confirmation unlock flow exists in phase 1.

### Lifecycle And Delete Behavior
Phase 1 does not expose public delete endpoints. Normal Player contracts must treat `soft-deleted` and `purge-pending` conversations as unavailable, while internal or admin-managed tooling may trigger delete and purge.

### Retention And Derived State
Conversation history is retained for 12 months after last activity unless deletion happens earlier. Derived memory and relationship state remain owner-bound to the conversation and are purged within 30 days after confirmed deletion. Audit-minimum traces follow their separate 90-day retention rule.

## Open Questions
No open questions are currently blocking the contract baseline.

## Recommendation Summary
Recommendation:
Use these contracts as the planning baseline for React browser, Go backend, Go worker, and internal lifecycle implementation. Keep the Player-facing API small, make delayed replies explicit through persisted scheduling, keep relationship scores hidden in phase 1, and route deletion through privileged internal flows rather than Player self-service.
