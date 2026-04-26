# ADR-0001 - Phase 1 Hosted Model Adult-Content Fallback

## Document Control
- Status: Accepted ADR
- Version: v1.1.0
- Last Updated: 2026-04-26
- Owner: SA

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.1.0 | Major | Converted the hosted-model adult-content fallback into the accepted phase 1 content posture: hosted public model operation is non-explicit only, with explicit adult sexual content deferred to a later local-model phase. | Architecture, API, implementation planning, and policy enforcement must align to the non-explicit hosted-model baseline and remove phase 1 adult-content enablement assumptions. |
| 2026-04-26 | v1.0.0 | Major | Established the hosted-model adult-content fallback ADR as a managed architecture decision baseline. | Architecture and implementation planning should continue to treat this ADR as a required constraint for phase 1 hosted-model behavior. |

## Upstream Baseline
- Based On: Phase 1 MVP PRD v3.0.2

## Context
The current PRD baseline requires phase 1 to run on a hosted public model such as Gemini, requires login before chat, and explicitly removes adult sexual content from the hosted-model MVP. Romance and friendship progression remain in scope, but explicit adult sexual content is deferred to a later local-model phase and requires separate approval before release.

This ADR exists because the earlier product direction left adult-content behavior conditional on provider feasibility. That conditional state no longer exists. The product baseline is now explicit:
- phase 1 hosted-model operation is non-explicit only
- no phase 1 Player-facing feature depends on explicit adult sexual content
- any future explicit-content capability belongs to a later local-model phase with a separate policy approval path

## Decision Drivers
- Keep phase 1 behavior consistent with the approved PRD baseline.
- Avoid building provider-dependent product behavior that the MVP no longer promises.
- Reduce policy and moderation complexity during the first release.
- Preserve a clean later migration path for stronger content control under a local model.

## Decision
Phase 1 uses a hosted public model in a non-explicit romance mode only.

The accepted architecture posture is:
- romance and friendship progression are supported in phase 1
- phase 1 MC behavior must remain non-explicit for all Players
- no phase 1 endpoint, UI flow, or prompt path should attempt to unlock explicit adult sexual content
- the provider adapter and policy layer may retain a future-facing capability seam, but only the non-explicit mode is enabled in phase 1
- a later local-model phase may revisit explicit adult sexual content only after a separate age-gating, safety, and policy approval decision

## Consequences
### Positive
- Phase 1 policy behavior is simple and unambiguous.
- Hosted-model provider constraints no longer create a product mismatch at runtime.
- API and UI scope shrink because no phase 1 explicit-content unlock flow is required.
- Safety testing focuses on non-explicit romance boundaries instead of dual-mode behavior.

### Negative
- Some Players may expect more explicit romance behavior than the MVP provides.
- A future local-model phase will still need a dedicated content-policy design pass.

### Neutral
- MC profile metadata can still preserve age, persona, and relationship boundaries.
- Provider portability remains important even though the immediate content posture is now fixed.

## Implementation Implications
- Remove any phase 1 planning assumption that a hosted provider might deliver approved explicit adult sexual content.
- Do not require a Player-facing age-gating unlock flow for phase 1 explicit-content access, because explicit-content access does not exist in phase 1.
- Enforce the hosted-model non-explicit posture inside the policy-safety module and provider orchestration path.
- Keep later local-model explicit-content support out of the phase 1 critical path.

## Decision Outcome
Accepted.

This ADR is now the source-of-truth architecture interpretation of the PRD rule that phase 1 hosted-model operation is non-explicit only.
