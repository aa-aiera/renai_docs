# ADR-0001 - Phase 1 Hosted Model Adult-Content Fallback

## Status
- Proposed

## Date
- 2026-04-26

## Context
The MVP PRD allows sexual content for players who confirm they are 18+ and interact with adult characters. However, the phase 1 technical plan also requires a hosted public model such as Gemini. Hosted public model policies may restrict or block the intended adult-content behavior even when the product itself permits it.

This creates a system-design risk:
- the product policy expects one behavior
- the hosted model provider may enforce a stricter one
- the MVP must not depend on unsupported behavior to function

The architecture packet already identifies this as an open question and recommends a policy capability switch.

## Decision Drivers
- phase 1 must remain deployable on a hosted public model
- the MVP core loop must still work even if explicit adult-content behavior is blocked
- the product must avoid a redesign when moving to a local LLM later
- safety and provider-policy failures must degrade gracefully

## Considered Options
### Option A: Assume Hosted Provider Supports Intended Adult Content
Pros:
- aligns most closely with the current product preference
- no visible behavior restriction for confirmed 18+ players

Cons:
- high risk of provider-policy mismatch
- fragile MVP planning
- could force late-stage product behavior changes

Assessment:
Rejected as the default planning assumption because it is too dependent on external policy.

### Option B: Phase 1 Runs In Hosted-Safe-Mode Only
Description:
Phase 1 allows romance and friendship progression, but any explicit adult sexual content is suppressed while the system is on a hosted public model. Adult-content capability is deferred until provider feasibility is proven or a local model is adopted.

Pros:
- safest operational default
- fully compatible with the current architecture
- avoids provider-policy breakage in the MVP
- preserves the core product loop

Cons:
- product behavior is narrower than the long-term vision
- some 18+ content expectations are deferred

Assessment:
Recommended.

### Option C: Move To Local LLM In Phase 1 To Preserve Adult Content
Pros:
- more direct control over policy behavior
- better alignment with unrestricted content goals

Cons:
- conflicts with the approved phase 1 hosted-model plan
- increases operational and model-serving complexity
- likely slows MVP delivery

Assessment:
Rejected for phase 1.

## Proposed Decision
Use `hosted-safe-mode` as the default and mandatory phase 1 operating mode.

This means:
- the MVP may still collect a simple 18+ confirmation dialog
- the product may still support romance progression for adult players
- explicit adult sexual content must be treated as not guaranteed in phase 1
- if the hosted model blocks or restricts that content, the system falls back to non-explicit romance and friendship behavior instead of failing the conversation

## Consequences
### Positive
- phase 1 remains viable on a hosted public model
- architecture remains stable
- provider-policy mismatch becomes a controlled fallback, not a delivery blocker
- local LLM migration in phase 2 remains a clean upgrade path

### Negative
- the phase 1 user experience for confirmed 18+ players may be narrower than originally desired
- product messaging must avoid promising explicit adult-content behavior that phase 1 cannot reliably deliver

## Implementation Impact
The system should implement a policy capability flag with at least these modes:
- `hosted-safe-mode`
- `adult-enabled-mode`

Phase 1 default:
- `hosted-safe-mode`

Phase 2 possible path:
- enable `adult-enabled-mode` only when provider feasibility or local-model control is confirmed

### Required Product Behavior In Hosted-Safe-Mode
- no explicit adult sexual content is relied on for core progression
- romance progression can continue in non-explicit form
- refusal or soft-redirection must remain in-character and policy-compliant

## Follow-Up Actions
1. Reflect this fallback in implementation policy configuration.
2. Keep explicit adult-content handling out of phase 1 acceptance criteria unless provider feasibility is confirmed.
3. Re-evaluate this ADR when the local LLM phase becomes active.

## Related Documents
- `docs/01_requirements/renai-game-llm-prd.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md`
- `docs/03_implementation/renai-game-llm-mvp-implementation-blueprint.md`
- `docs/03_implementation/renai-game-llm-mvp-api-contract-drafts.md`