# Renai Game LLM MVP - API Contract Drafts

## Document Control
- Status: Draft API Contracts
- Version: v1.0
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.0 | Major | Established the API contract drafts as a managed implementation-planning artifact. | Implementor execution should treat these contracts as part of the Planning v1.0 baseline until the architecture or planning packet changes. |

## Upstream Baseline
- Based On: MVP PRD v1.0 and phase 1 architecture packet v1.0

## Executive Summary
This document defines draft API contracts for the phase 1 MVP browser client, backend API, and async worker job payloads. The contracts are intentionally transport- and stack-neutral at the schema level, but they assume a conventional HTTP JSON API between the browser and backend and queue-delivered JSON payloads between the backend and worker.

These contracts are designed for:
- guest trial before login
- Facebook-based returning identity
- one-on-one conversation restore
- delayed character replies
- conversation-scoped memory updates
- hidden internal relationship scores in phase 1

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
Use `/v1/` for all MVP endpoints.

### Content Type
- Request: `application/json`
- Response: `application/json`

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
- guest requests carry a guest session context
- authenticated requests carry a resolved user account context
- guest state resets on login and is not migrated

### Lifecycle And Retention Semantics
- guest sessions are disposable and may return an `expires_at` value
- conversations may move through lifecycle states such as `active`, `inactive`, `soft-deleted`, and `purge-pending`
- non-active conversations are not returned to normal client flows and do not accept new writes or delayed replies
- derived memory and relationship state follow the owner conversation lifecycle and are not exposed as independent client resources

## HTTP API Contracts
### 1. App Bootstrap / Guest Session Create
`POST /v1/session/guest`

Purpose:
Create or refresh a guest session for pre-login use.

Request:
```json
{
  "client_context": {
    "locale": "th-TH",
    "timezone": "Asia/Bangkok"
  }
}
```

Response:
```json
{
  "data": {
    "guest_session_id": "uuid",
    "guest_input_limit": 10,
    "guest_input_count": 0,
    "expires_at": "2026-04-27T10:00:00Z",
    "age_confirmation": {
      "confirmed_18_plus": false,
      "confirmed_at": null
    }
  },
  "meta": {},
  "error": null
}
```

### 2. Set Age Confirmation
`POST /v1/session/age-confirmation`

Purpose:
Record self-attested 18+ confirmation for the current guest or authenticated session.

Request:
```json
{
  "confirmed_18_plus": true
}
```

Response:
```json
{
  "data": {
    "confirmed_18_plus": true,
    "confirmed_at": "2026-04-26T10:00:00Z"
  },
  "meta": {},
  "error": null
}
```

### 3. Character List
`GET /v1/characters`

Purpose:
Load the MVP roster for discovery screens.

Response:
```json
{
  "data": [
    {
      "character_id": "uuid",
      "code_name": "aira",
      "display_name": "Aira",
      "profile_summary": "Warm, witty, observant",
      "avatar_url": "string-or-null"
    }
  ],
  "meta": {
    "count": 3
  },
  "error": null
}
```

### 4. Character Detail
`GET /v1/characters/{character_id}`

Purpose:
Load detailed character profile information for discovery and chat context.

Response:
```json
{
  "data": {
    "character_id": "uuid",
    "display_name": "Aira",
    "age_years": 22,
    "adult_character": true,
    "persona_summary": "Warm, witty, observant",
    "talking_style_summary": "Playful teasing, casual Thai tone",
    "interests_summary": "Art, music, campus life, coffee",
    "relationship_boundary_summary": "Romance requires trust and comfort"
  },
  "meta": {},
  "error": null
}
```

### 5. Facebook Auth Exchange
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
    "user_account_id": "uuid",
    "session_type": "authenticated",
    "guest_state_migrated": false,
    "age_confirmation": {
      "confirmed_18_plus": false,
      "confirmed_at": null
    }
  },
  "meta": {},
  "error": null
}
```

### 6. Get Or Create Conversation For Character
`POST /v1/conversations/by-character/{character_id}`

Purpose:
Resolve the active conversation root for the current identity and selected character.

Response:
```json
{
  "data": {
    "conversation_id": "uuid",
    "character_id": "uuid",
    "owner_scope": "guest-or-authenticated",
    "conversation_status": "active"
  },
  "meta": {},
  "error": null
}
```

### 7. Load Conversation State
`GET /v1/conversations/{conversation_id}`

Purpose:
Return the chat state needed to render the conversation view.

Response:
```json
{
  "data": {
    "conversation_id": "uuid",
    "conversation_status": "active",
    "character": {
      "character_id": "uuid",
      "display_name": "Aira"
    },
    "messages": [
      {
        "message_id": "uuid",
        "sender_type": "user",
        "message_text": "hello",
        "delivery_state": "delivered",
        "created_at": "2026-04-26T10:00:00Z"
      }
    ],
    "guest_trial": {
      "is_guest": true,
      "guest_input_limit": 10,
      "guest_input_count": 3,
      "login_required": false
    }
  },
  "meta": {},
  "error": null
}
```

### 8. Send User Message
`POST /v1/conversations/{conversation_id}/messages`

Purpose:
Accept a user message, persist it, and schedule a delayed character reply.

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
      "sender_type": "user",
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

Guest Limit Reached Response:
```json
{
  "data": null,
  "meta": {
    "login_required": true
  },
  "error": {
    "code": "GUEST_LIMIT_REACHED",
    "message": "Login is required to continue chatting."
  }
}
```

### 9. Poll Latest Conversation Messages
`GET /v1/conversations/{conversation_id}/messages?after={message_id}`

Purpose:
Allow the client to refresh newly delivered character replies.

Response:
```json
{
  "data": [
    {
      "message_id": "uuid",
      "sender_type": "character",
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

### 10. List User Conversations
`GET /v1/me/conversations`

Purpose:
Return the authenticated user’s conversation list for chat restore and resume.

Response:
```json
{
  "data": [
    {
      "conversation_id": "uuid",
      "character_id": "uuid",
      "character_display_name": "Aira",
      "last_activity_at": "2026-04-26T10:05:03Z",
      "last_message_preview": "I just got back from class."
    }
  ],
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
  "character_id": "uuid",
  "owner_scope": "guest-or-authenticated"
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
  "user_message_id": "uuid",
  "character_message_id": "uuid"
}
```

### 5. Emit Audit Event Job
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
| `GUEST_LIMIT_REACHED` | Guest user hit the 10-input cap |
| `AGE_CONFIRMATION_REQUIRED` | 18+ confirmation must be completed before gated behavior |
| `SESSION_EXPIRED` | Guest or authenticated session is expired or invalid |
| `CONVERSATION_NOT_FOUND` | Conversation is missing or does not belong to caller |
| `CONVERSATION_UNAVAILABLE` | Conversation exists but is deleted, purge-pending, or otherwise inaccessible |
| `POLICY_BLOCKED` | Request blocked by hard safety rules |
| `PROVIDER_UNAVAILABLE` | Hosted LLM call failed or timed out |
| `REPLY_NOT_READY` | Delayed reply exists but is not yet delivered |

## Contract Notes
### Phase 1 Score Visibility
The API should not expose relationship scores to players in phase 1.

### Guest Reset Behavior
The auth exchange contract explicitly returns `guest_state_migrated: false` so this product rule is visible to the client.

### Lifecycle And Delete Behavior
Phase 1 does not require public delete endpoints, but lifecycle-aware contracts should ensure that expired guest sessions and soft-deleted or purge-pending conversations stop accepting new writes and stop receiving delayed replies.

### Adult-Content Policy Risk
The contracts allow age-confirmation state, but they do not guarantee hosted-provider support for the intended adult-content behavior. The current planning assumption follows the phase 1 `hosted-safe-mode` ADR unless provider feasibility is confirmed.

## Open Questions
1. Can the phase 1 hosted model and provider policy support the intended adult-content policy for players who confirm they are 18+, or must adult sexual content wait for a later phase or local model?
2. What concrete retention durations should map to guest-limited, account-durable, and audit-minimum lifecycle classes?
3. Should phase 1 expose any user-facing delete endpoint, or should delete remain an internal or admin-managed lifecycle path until later?

## Recommendation Summary
Recommendation:
Use these contracts as draft engineering interfaces for browser, backend, and worker implementation. Keep the browser API simple, make delayed replies explicit through persisted scheduling, and avoid exposing relationship scores in phase 1.