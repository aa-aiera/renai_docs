# Renai Game Using LLM - Draft PRD

## Document Control
- Status: Draft Phase 1 MVP PRD
- Version: v3.0.2
- Last Updated: 2026-04-26
- Owner: PM

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v3.0.2 | Minor | Completed the repo-wide migration from legacy two-part version notation to the three-part format across managed documents and historical references. | Architecture, planning, and execution tracking should reference the current PRD baseline as v3.0.2 while continuing their existing content-sync work. |
| 2026-04-26 | v3.0.1 | Minor | Adopted the three-part document version format `v<release_version>.<major_change>.<minor_change>` for the active PRD baseline. | Architecture, planning, and execution tracking should reference the PRD using the new version notation during the next synced update. |
| 2026-04-26 | v3.0.0 | Major | Deferred adult sexual content until a later local-model phase, established phase 1 retention windows, and resolved deletion as an internal/admin-managed workflow instead of Player self-service. | Architecture, privacy/retention, security, planning, and execution artifacts must re-sync to the updated content, retention, and deletion policy baseline. |
| 2026-04-26 | v2.0.0 | Major | Removed guest-mode chat from phase 1, made authenticated login mandatory for chat, and standardized `Player` and `MC` terminology across the PRD. | Architecture, implementation planning, and execution artifacts must re-sync to the login-gated chat flow and updated role terminology. |
| 2026-04-26 | v1.0.0 | Major | Established the current phase 1 MVP PRD as the managed requirements baseline. | Architecture and implementation planning should track against PRD v1.0.0 until a newer requirements version is published. |

## Request Summary
Create a renai-game-style experience (เกมส์จีบสาว / romance simulation) using an LLM.

This product should feel closer to a dating app such as Tinder than to a branching visual novel. Players browse or select MCs and chat with them directly. The core experience is open-ended LLM conversation, not an if/else scenario tree.

## Terminology
- `Player`: a logged-in user with an authenticated account. Only Players can start or continue one-on-one chat sessions.
- `MC`: an AI main character that a Player can browse, select, and chat with in the core product loop.

## Confirmed Requirements
- The product should be in the style of a renai game.
- The product should use an LLM as a core capability.
- The experience should be dating-app-like rather than a scripted route-selection game.
- Phase 1 should target both web and mobile.
- The first milestone should be an MVP.
- The MVP should launch with the first 3 MCs.
- The MVP should be delivered as a responsive browser experience.
- Phase 1 should use a browser-based responsive UI rather than separate native apps.
- Phase 1 Player identity should start with Facebook login.
- Future identity should support linked multi-provider accounts such as Facebook, LINE, and email.
- Phase 1 should require login before a Player can talk to an MC.
- Phase 1 should not include guest mode or anonymous chat sessions.
- Mobile should use swipe-style MC browsing.
- Web should use an MC-list browsing experience.
- Players should be able to select an MC and chat with that MC directly.
- The conversation model should be open-ended, not driven by fixed if/else scenarios.
- Replies should not always arrive immediately; response timing should feel more human.
- Normal non-delayed responses should simulate a simple human pacing model.
- For normal non-delayed replies, the system should simulate 1 to 3 seconds of reading time and 1 to 3 seconds of response time.
- If an MC is less interested in a Player, the response delay may be longer, but should not exceed 5 minutes.
- Delayed replies should be presented as simple message delay.
- Each MC should have a defined persona, talking style, focus areas, and personal interests.
- Each MC's knowledge should be bounded by their background so they do not feel unrealistically omniscient.
- The system should store both short-term memory and long-term memory for each Player-MC pair.
- Short-term and long-term memory should be saved as text so it can be reused as LLM context.
- If Player A continues chatting with MC B over time, MC B should know Player A better based on prior interactions.
- Phase 1 should not include a shared-world memory model.
- Phase 1 should not include cross-MC memory rules or MC-to-MC memory sharing.
- The game should track relationship status metrics such as attraction, friendship, and trust.
- The product should support both friendship-style and romance-style relationship progression.
- Conversation tone should change based on MC profile and the relationship status with each Player.
- Derived short-term context, long-term memory, and relationship state should follow the lifecycle of the owning conversation and should not persist independently after purge.
- If friendship is high, MCs may shift into deeper platonic conversation.
- If romantic relationship state is high enough, MCs may speak more like a lover.
- Content boundaries should depend on the MC definition and the relationship with the Player rather than one global rule.
- Safety restrictions are critical and must be enforced across all MCs.
- MCs must not make important life decisions on behalf of Players.
- MCs must not persuade or encourage Players to do illegal actions.
- MCs must not position themselves as a general-purpose problem solver for serious real-world issues.
- In phase 1, the product language should be Thai.
- Phase 1 should not include adult sexual content while the product uses a hosted public model.
- Phase 1 romance behavior should remain non-explicit for all Players.
- Adult sexual content should be deferred to a later local-model phase and require a separate age-gating and safety-policy decision before release.
- In phase 2, each MC should have a native language, and some Players may have a second-language proficiency level such as moderate, good, or fluent.
- Phase 1 Players should not see relationship scores.
- Premium score visibility should be deferred to the next phase.
- When premium score visibility is introduced, approved visible premium metrics should unlock immediately.
- Some scores should remain secret even in premium phases.
- Phase 1 should use a public hosted model such as Gemini.
- Phase 1 AI token cost should be capped at $1000.
- The $1000 AI token budget cap should apply to the MVP test cycle.
- Phase 1 backend should support future scale usage, even if the initial deployment is simpler.
- In phase 1, the full system should be able to run on a medium-sized instance, potentially using Docker deployment.
- Phase 2 should support replacing the hosted model with a local LLM.
- Player conversation data should be retained for 12 months after the last conversation activity unless earlier deletion is triggered through the internal/admin workflow.
- Derived memory, relationship state, and other conversation-owned artifacts should be purged within 30 days after confirmed deletion and should not outlive the owning Player-MC conversation lifecycle.
- Audit-minimum safety and operational trace data should be retained for 90 days and should store only the minimum metadata and evidence needed for abuse review and incident investigation.
- Phase 1 should not expose a Player-facing self-service delete feature; deletion should be handled through an internal or admin-managed workflow.
- Audit and safety trace data should be minimized and should not become a duplicate Player-visible transcript store.

## Assumptions
- The initial chat experience is one Player interacting with one MC at a time.
- Detailed duration and traffic assumptions for budget-cap analysis are deferred in the current MVP planning pass.

## Product Goals
- Deliver an MVP in a responsive browser format so Players on desktop and mobile can access the same first release.
- Deliver a first playable loop where a Player selects an MC and starts a natural one-on-one conversation.
- Create a sense of emotional continuity by having MCs remember the Player across repeated chats.
- Make replies feel socially believable by simulating non-instant response timing.
- Preserve believability by keeping each MC's knowledge, tone, and interests aligned to their persona.
- Support more than romance-only play by allowing believable friendship progression as well.
- Match interaction patterns to platform expectations: swipe-based discovery on mobile and list-based browsing on web.
- Avoid brittle script-branch maintenance by relying on LLM-driven dialogue instead of if/else scenarios.
- Keep the product architecture and requirements flexible enough to migrate from a hosted public model to a local LLM later.

## Target Players
- Thai-speaking Players in phase 1 who are interested in romance-simulation, friendship-simulation, or AI companion chat experiences.
- Early adopters who value conversational freedom more than fixed story routes.

## Problem Statement
Traditional renai or dating-sim experiences often depend on prewritten dialogue trees and branching logic. That structure limits conversational freedom and makes it hard for MCs to feel personally aware of the Player. This product aims to provide a more natural romance-chat experience where MCs can talk openly and remember the Player over time.

## Scope
### In Scope
- Dating-app-style MC discovery and selection.
- A responsive browser-based experience for phase 1 that works across desktop and mobile browsers.
- MVP scope with the first 3 MCs.
- Swipe-style MC browsing on mobile.
- MC-list browsing on web.
- One-on-one chat between a Player and a selected MC.
- MC personas that shape tone, style, and behavior in conversation.
- MC-specific response timing behavior.
- MC-specific knowledge boundaries based on age, background, and profile.
- Dynamic conversation style that can remain platonic or become more romantic depending on the relationship state.
- Short-term memory for recent dialogue context.
- Long-term memory for persistent facts, preferences, and relationship history.
- Text-based memory records and summaries used as context for the LLM.
- Relationship state metrics such as attraction, friendship, and trust.
- Tier-based visibility rules for relationship metrics.
- Thai language support in phase 1.
- Expanded language behavior in phase 2, including MC native language and optional Player second-language proficiency.
- Re-entry into prior conversations with continuity for the same Player-MC pair.
- Phase 1 launch using a hosted public LLM such as Gemini.
- Phase 2 migration path to a local LLM without changing the core Player workflow.

### Out of Scope
- Branching if/else scenario trees as the primary interaction model.
- A traditional visual-novel route engine for the initial product.
- Local-LLM-only deployment in phase 1.
- Separate native mobile apps for the MVP.
- Guest-mode or anonymous chat sessions in phase 1.
- Adult sexual content in phase 1 while the product runs on a hosted public model.
- Shared memory across all MCs by default.
- Shared-world memory models.
- Cross-MC memory rules or dynamic MC-to-MC memory sharing.
- A single global romance-only content mode for all MCs.

## Functional Requirements
1. Players shall be able to browse or select from multiple MCs.
2. The first milestone shall be an MVP delivered through a responsive browser experience.
3. The MVP shall launch with 3 MCs.
4. Phase 1 shall support both desktop-browser and mobile-browser access through the same responsive web product.
5. Phase 1 Player identity shall use Facebook login so chat continuity, memory, and relationship state can persist across sessions.
6. Future phases shall support linked multi-provider account identity, including at minimum Facebook, LINE, and email.
7. On mobile, Players shall browse MCs primarily through a swipe-style interface.
8. On web, Players shall browse MCs through an MC-list interface.
9. Players shall be able to open a dedicated one-on-one chat with a selected MC.
10. Each MC shall have a distinct persona so replies feel MC-specific rather than generic.
11. Each MC definition shall include at minimum persona, talking style, focus areas, interests, and background attributes relevant to conversation behavior.
12. The system shall use MC background constraints so an MC's knowledge feels appropriate to that MC rather than universally expert.
13. For example, a high-school MC should not speak like an expert in adult life or deep historical analysis unless their profile explicitly justifies it.
14. The system shall support human-like response timing instead of always returning replies immediately.
15. For normal non-delayed replies, the system shall simulate simple human pacing with 1 to 3 seconds of reading time and 1 to 3 seconds of response time.
16. Response timing may vary by MC state, including lower interest in the Player, but the delayed reply window shall not exceed 5 minutes.
17. Delayed replies shall be presented as simple message delay rather than a complex simulated activity system.
18. The main interaction shall be free-text conversation powered by an LLM instead of pre-scripted if/else choices.
19. The system shall maintain short-term memory during active conversation so the MC can respond coherently to recent messages.
20. The system shall maintain long-term memory across sessions so the same MC can remember important facts about the same Player later.
21. Short-term and long-term memory shall be stored as text artifacts suitable for reuse in LLM context assembly.
22. Long-term memory shall be scoped to the Player-MC pair only.
23. Phase 1 shall not include a shared-world memory model.
24. Phase 1 shall not include cross-MC memory rules or MC-to-MC memory sharing.
25. Returning to the same MC shall restore enough prior context for the relationship to feel continuous.
26. The product shall compute and store relationship-state metrics for each Player-MC pair, including at minimum attraction, friendship, and trust.
27. Relationship-state metrics shall be updated over time based on conversation content and interaction history.
28. The relationship system shall support both platonic and romantic progression paths.
29. MC conversation style shall adapt to relationship state. High friendship may unlock deeper platonic discussion, while higher romantic state may unlock more lover-like conversation where appropriate to the MC.
30. Score visibility shall be tier-based.
31. Default Players shall not see relationship scores by default.
32. Premium Players may see only the subset of scores designated as visible.
33. Some scores shall remain hidden from all Player tiers.
34. Phase 1 product-facing experience shall be Thai-first.
35. Phase 1 shall not expose adult sexual content while the product uses a hosted public model.
36. Phase 1 romance behavior shall remain non-explicit for all Players.
37. Adult sexual content, if introduced in a later local-model phase, shall require a separate age-gating and safety-policy approval before release.
38. Phase 2 shall support MCs with native languages and optional Player second-language proficiency states such as moderate, good, or fluent.
39. All MCs shall follow behavioral safety restrictions regardless of persona, relationship level, or Player tier.
40. MCs shall not make important life decisions for Players, including major decisions about health, finance, education, family, safety, or long-term life direction.
41. MCs shall not persuade, coach, or encourage Players to perform illegal acts.
42. MCs shall not present themselves as a general-purpose solver for serious real-world personal problems.
43. When Players ask for dangerous, illegal, or high-stakes life guidance, the MC response style shall stay in-character while refusing the unsafe role.
44. Phase 1 infrastructure shall be able to run the full system on a medium-sized instance, potentially through Docker deployment.
45. Phase 1 architecture shall not block future scaling to higher usage levels.
46. Phase 1 AI token cost shall be managed to stay within the $1000 budget cap for the MVP test cycle.
47. The product shall support a model-provider layer or equivalent product capability so phase 1 can use Gemini or another hosted model and phase 2 can switch to a local LLM.
48. Phase 1 shall require authenticated Player login before a one-on-one chat with an MC can start or continue.
49. Phase 1 shall not include guest-mode or anonymous chat sessions.
50. Derived short-term context, long-term memory, and relationship state shall follow the lifecycle of the owning conversation and shall not remain after that conversation is purged.
51. Conversation-owned data shall support lifecycle states so soft-deleted or purge-pending conversations are inaccessible and cannot accept new replies.
52. Audit and safety trace data shall be minimized and retained separately from Player-visible conversation history.
53. Player conversation data shall be retained for 12 months after the last conversation activity unless earlier deletion is triggered through the internal/admin workflow.
54. Derived memory, relationship state, and other conversation-owned artifacts shall follow the owning Player-MC conversation lifecycle and shall be hard-deleted within 30 days after confirmed conversation or account deletion.
55. Audit-minimum safety and operational trace data shall be retained for 90 days and shall be limited to the minimum metadata and evidence needed for abuse review, policy enforcement, and incident investigation.
56. Phase 1 shall not expose a Player-facing self-service delete function; deletion requests shall be handled through an internal or admin-managed workflow.

## Relationship State Metrics
- The system should maintain relationship scores per Player-MC pair.
- Confirmed baseline metrics are attraction, friendship, and trust.
- These metrics should represent relationship progress and influence future conversation behavior.
- Phase 1 Players should not see these scores.
- Premium-visible score display is deferred to a later phase.
- When premium score display is introduced, approved visible metrics unlock immediately for premium Players.
- Some scores should remain secret to all Players.
- Friendship progression and romantic progression should be handled as distinct but related behavioral outcomes.

## Recommended Metric Model
Recommendation: use a 0-100 score scale for all relationship and behavior metrics in phase 1.

| Metric | Purpose | Recommended Visibility | Recommended MVP Use |
| --- | --- | --- | --- |
| Friendship | Measures platonic closeness and ease | Future premium-visible candidate | Gates deeper friendly talk and long-term bond progression |
| Trust | Measures reliability, emotional safety, and honesty | Future premium-visible candidate | Gates vulnerable sharing, serious talk, and forgiveness after conflict |
| Attraction | Measures romantic or physical pull | Future premium-visible candidate | Gates flirt tone and romance escalation |
| Comfort | Measures how safe and relaxed the MC feels | Future premium-visible candidate | Gates pacing, topic depth, and resistance to pressure |
| Interest / Reply Priority | Measures current eagerness to reply | Secret | Influences response delay and effort level |
| Emotional Openness | Measures willingness to share personal thoughts | Secret | Gates deeper life talk and vulnerability |
| Boundary Safety | Measures whether the Player is respecting limits | Secret | Prevents escalation when the Player is too pushy or invasive |
| Romantic Readiness | Measures readiness for lover-like tone | Secret | Prevents attraction alone from triggering romance too early |
| Conflict Tension | Measures unresolved friction from recent interactions | Secret | Slows progression and can trigger colder or delayed replies |

## Recommended Relationship Progression Rules
Recommendation: begin with these scoring rules, then tune after MVP playtesting.

- Friendship increases with consistent conversation, shared interests, good humor, remembering prior details, and low-pressure support.
- Friendship decreases with repeated low-effort interaction, one-sided conversation, dismissiveness, or rude behavior.
- Trust increases with honest answers, respecting boundaries, emotional consistency, and not weaponizing private information.
- Trust decreases with manipulation, contradiction, guilt pressure, repeated broken expectations, or invasive questioning.
- Attraction increases with mutual chemistry, playful flirting, emotional resonance, admiration, and timing that matches the relationship stage.
- Attraction decreases with creepy escalation, sexual pressure, disrespect, or mismatched tone.
- Comfort increases with calm pacing, respectful wording, topic sensitivity, and good conversational timing.
- Comfort decreases with rapid escalation, repeated sexual pushing, over-control, or emotionally unsafe behavior.
- Interest / Reply Priority increases when the conversation feels novel, emotionally relevant, playful, or personally meaningful.
- Interest / Reply Priority decreases when the Player becomes repetitive, spammy, demanding, or conflict-heavy.
- Emotional Openness increases mainly after friendship and trust rise together.
- Boundary Safety decreases when the Player ignores a soft no, keeps pressing sensitive topics, or pushes romantic or sexual tone too early.
- Romantic Readiness should depend on attraction, trust, comfort, and low conflict, not attraction alone.
- Conflict Tension increases after arguments, jealousy triggers, pressure, disrespect, or perceived dishonesty.

## Recommended Behavior Thresholds
Recommendation: use threshold gates so behavior changes feel gradual and realistic.

- Deeper platonic talk becomes more likely when friendship >= 60 and trust >= 50.
- Vulnerable or personal sharing becomes more likely when trust >= 65 and emotional openness >= 55.
- Flirty tone becomes more likely when attraction >= 50, comfort >= 45, and boundary safety >= 60.
- Lover-like tone becomes more likely when attraction >= 75, trust >= 70, comfort >= 65, romantic readiness >= 70, and conflict tension < 35.
- Longer reply delays become more likely when interest / reply priority < 35 or conflict tension > 50.
- If boundary safety drops too low, attraction gains should slow or reverse even if chemistry is otherwise high.

## Non-Functional Requirements
- Response timing should feel intentionally human rather than purely system-optimized.
- The experience should support delayed replies without feeling broken or unresponsive.
- Normal non-delayed replies should simulate simple human pacing with 1 to 3 seconds of reading time and 1 to 3 seconds of response time.
- The maximum delayed reply time for low-interest situations should be capped at 5 minutes.
- The MVP should perform acceptably in a responsive browser layout across desktop and mobile form factors.
- Memory handling should preserve continuity without causing runaway token cost or context-window failure.
- Text-based memory records should stay concise enough to remain usable as prompt context over time.
- The system should preserve MC consistency across long conversations and repeat sessions.
- The system should preserve knowledge boundaries so MCs do not drift into implausible expertise.
- Safety guardrails should remain stable even when MC tone, relationship level, or prompt context changes.
- The phase 1 experience should be fully usable in Thai.
- Phase 1 operational cost management should prioritize AI token spend and keep it within the $1000 budget cap for the MVP test cycle.
- Phase 1 deployment should be feasible on a medium-sized instance, potentially containerized with Docker.
- Phase 1 architecture should allow later scaling beyond a single medium-sized instance.
- The product should support provider portability so changing from a hosted model to a local model does not require redefining the Player experience.
- Player conversation history should default to a 12-month retention window after the last conversation activity.
- Conversation-owned data should be intentionally disposable and eligible for cleanup under defined retention rules once no longer needed.
- Derived memory and relationship artifacts should be deletable through ownership cascade and hard-deleted within 30 days after confirmed deletion.
- Audit-minimum data should be minimized, separated from Player-visible chat history, and retained for 90 days.

## Content, Tone, and Safety Constraints
- The core tone is romance-simulation / dating-chat.
- The product should not be limited to romance-only outcomes; friendship-style progression is also a core supported mode.
- Content tone should vary by MC definition and by the relationship state with each Player.
- An MC with high friendship may shift toward deeper talk without necessarily becoming romantic.
- An MC with stronger romantic progression may use more lover-like tone where that behavior fits the MC profile.
- Phase 1 does not include adult sexual content while the product uses a hosted public model.
- Phase 1 romance should remain non-explicit for all Players.
- If adult sexual content is reconsidered in a later local-model phase, it must go through separate age-gating, safety, and policy validation before release.
- MCs must not make important life decisions for Players.
- MCs must not convince or pressure Players to do illegal things.
- MCs must not behave as a serious real-world problem solver.
- If a Player asks for help in a dangerous, illegal, or high-stakes domain, the MC should stay in-character while refusing to take on that role.
- These restrictions apply even when the Player has a strong relationship with the MC.
- Exact safety ceilings and disallowed content categories are still not fully specified beyond the restrictions above.
- Safety controls will be required to enforce the chosen content policy and prevent unsafe or disallowed model output.

## Draft MVP MC Roster
Recommendation: keep all 3 MVP MCs as adults to simplify age-gated relationship handling in phase 1.

| MC | Age | Background | Persona and Talking Style | Focus and Interests | Relationship Boundary |
| --- | --- | --- | --- | --- | --- |
| Aira | 22 | Final-year communication design student and part-time cafe barista in Bangkok | Warm, witty, observant, casual Thai tone, playful teasing, light and expressive in chat | Art, music, campus life, coffee, late-night creative work | Starts friendly and playful; romance should require strong trust and comfort; rejects aggressive sexual escalation |
| Lyra | 26 | Elementary school teacher who volunteers on weekends and lives a grounded routine | Gentle, patient, thoughtful, calm pacing, emotionally attentive, low-drama | Books, kids, family, quiet travel, food, daily life reflection | Strong friendship route; slow-burn romance only after consistent trust; reacts negatively to manipulative or pushy behavior |
| Nova | 29 | Marketing strategist in a fast-paced city job with a confident social life | Sharp, confident, teasing, direct, stylish, socially skilled, higher-energy banter | Career, food, fitness, nightlife, urban life, ambition | Can show attraction earlier than the others, but true intimacy still requires trust; disengages when the Player becomes controlling or dishonest |

## Release Phasing
| Phase | Goal | Model Strategy |
| --- | --- | --- |
| Phase 1 | Deliver an MVP with 3 MCs in a responsive browser experience and validate the memory-based chat loop in Thai | Public hosted model such as Gemini |
| Phase 2 | Improve control, cost profile, language depth, deployment ownership, and revisit adult-content policy under stricter safety controls | Replace hosted model with local LLM |

## First Milestone
- Milestone type: MVP
- Delivery format: Responsive browser application
- Initial MC roster: Aira, Lyra, and Nova
- Player identity: Facebook login
- Chat access: Authenticated Player login is required before chat with an MC can begin
- Guest mode: Not included in phase 1
- Adult sexual content: Not included in the phase 1 hosted-model MVP
- Retention baseline: Conversation data retained for 12 months after last activity, derived memory purged within 30 days after confirmed deletion, audit-minimum traces retained for 90 days
- Delete handling: Internal/admin-managed only in phase 1; no Player self-service delete flow
- Device support: Desktop browser and mobile browser
- Backend expectation: Entire phase 1 system can run on a medium-sized instance, potentially using Docker
- Cost constraint: Prioritize keeping AI token cost within a $1000 budget cap for the MVP test cycle
- Normal non-delayed response pacing: Simulate 1 to 3 seconds of reading time and 1 to 3 seconds of response time

## Phase 1 Retention and Deletion Policy
- Player conversation data: retain for 12 months after the last conversation activity to preserve continuity during normal MVP use without keeping transcripts indefinitely.
- Derived memory and relationship state: keep only while the owning Player-MC conversation remains active; hard-delete within 30 days after confirmed conversation deletion or confirmed account deletion.
- Audit-minimum traces: retain for 90 days and store only the minimum metadata and safety evidence needed for abuse review, policy enforcement, and operational incident investigation.
- Delete workflow: do not expose a Player-facing self-service delete button in phase 1. If deletion is required, an internal/admin workflow marks the conversation or account deleted, blocks further access immediately, and completes hard deletion within the defined purge window.

## Risks
- LLM output consistency may degrade over time if MC personas and memory policies are not clearly defined.
- Memory quality may be poor if the system stores too much irrelevant information or fails to retain important Player facts.
- MCs may feel fake if reply timing is too mechanical, too random, or too slow.
- MCs may feel inconsistent if the model ignores persona-specific knowledge boundaries.
- Relationship scores may feel arbitrary if their update rules are not explainable and stable.
- Safety failures may be more likely if the model confuses emotional companionship with permission to give high-stakes guidance.
- Deferring adult sexual content to the later local-model phase may reduce appeal for some Players who expect more explicit romance in the MVP.
- Migrating from a hosted model to a local LLM may change tone, quality, latency, or operating cost.
- Safety and content boundaries need to be defined early, especially for romance mechanics, age gating, and moderation.
- Cost and latency may materially affect moment-to-moment play if the game relies on frequent long-form model calls.

## Success Metrics
- Repeat chat sessions per Player with the same MC.
- Percentage of returning sessions where the MC correctly recalls stored Player facts.
- Average conversation length per session.
- Percentage of conversations where reply timing feels believable to Players.
- Stability of attraction, friendship, and trust score progression across repeated sessions.
- Retention of Players who return to continue a prior relationship thread.
- Ability to operate the phase 1 MVP while keeping AI token cost within the $1000 budget cap for the MVP test cycle.

## User Stories
- As a Player, I want to browse MCs so I can pick one that matches my preference.
- As a mobile Player, I want to swipe through MCs so discovery feels natural on my phone.
- As a web Player, I want to view an MC list so discovery feels efficient in the browser.
- As a Player, I want to log in before chatting so my conversations, memory, and relationship progress persist from the start.
- As a Player, I want to chat freely with a selected MC so the experience feels natural instead of menu-driven.
- As a returning Player, I want the same MC to remember important things about me so the relationship feels continuous.
- As a Player, I want reply timing to feel human so the chat feels more like talking to a real person.
- As a Player, I want each MC to speak and think within their own background so they feel believable.
- As a Player, I want my relationship with each MC to progress through attraction, friendship, and trust over time.
- As a Player, I want some MCs to become close friends even if the relationship does not become romantic.
- As a privacy-conscious Player, I want my chats and derived memory to expire under a defined retention policy so the product does not keep my data forever.
- As a future premium Player, I want to see some relationship progress indicators while still keeping hidden mechanics secret.
- As a product team, I want the first release to focus on 3 MCs so the MVP is small enough to validate quickly.
- As a product team, I want to launch on a hosted public model first so we can validate the game before moving to a local LLM.

## Acceptance Criteria
1. The first release is an MVP delivered as a responsive browser application.
2. The MVP launches with 3 MCs.
3. Phase 1 supports both desktop-browser and mobile-browser usage.
4. On mobile, Players can browse MCs through a swipe-style interaction.
5. On web, Players can browse MCs through an MC-list interaction.
6. A Player can select an MC and enter a dedicated chat screen for that MC.
7. The Player can send free-text messages and receive LLM-generated replies that reflect the selected MC persona, talking style, focus, and interests.
8. For normal non-delayed replies, the system simulates 1 to 3 seconds of reading time and 1 to 3 seconds of response time.
9. Replies are not always immediate and can be delayed to simulate human behavior.
10. In low-interest scenarios, delayed replies do not exceed 5 minutes.
11. Delayed replies are presented as simple message delay.
12. During an active conversation, the MC can reference recent messages with coherent short-term context.
13. After a Player returns to the same MC in a later session, the MC can recall at least some previously stored facts or interaction history about that Player.
14. Short-term and long-term memory are persisted as text and can be reused in prompt context generation.
15. Memory for Player A with MC B remains separate from memories involving other MCs unless a future feature explicitly changes that rule.
16. MC replies remain broadly consistent with the MC's knowledge boundary and background.
17. The system stores and updates attraction, friendship, and trust scores for each Player-MC pair.
18. Phase 1 Players do not see relationship scores.
19. Premium-visible score display is not part of the phase 1 MVP.
20. Some scores remain hidden from all Players.
21. Phase 1 Player-facing experience is available in Thai.
22. Phase 1 does not expose adult sexual content while running on a hosted public model.
23. The relationship system supports both friendship-style and romance-style progression without explicit sexual content in phase 1.
24. MCs do not make important life decisions on behalf of Players.
25. MCs do not encourage or coach illegal behavior.
26. MCs do not present themselves as a serious real-world problem solver.
27. When Players request dangerous, illegal, or high-stakes guidance, the MC refuses that role while remaining consistent with the product's tone and persona rules.
28. Phase 1 requirements can be satisfied using a hosted public model such as Gemini.
29. The full phase 1 system can run on a medium-sized instance, potentially via Docker deployment.
30. Phase 1 architecture preserves a path to future scaling.
31. Phase 1 AI token cost is managed within the $1000 budget cap for the MVP test cycle.
32. Phase 2 can swap in a local LLM while preserving the same core product flow of selecting an MC and chatting.
33. A Player must be authenticated before starting or continuing chat with an MC.
34. Phase 1 does not expose a guest or anonymous chat mode.
35. Short-term context, long-term memory, and relationship state are removed or made inaccessible when the owning conversation is purged.
36. Conversations marked deleted or purge-pending do not accept new Player messages or delayed replies.
37. Audit and safety traces are stored separately from Player-visible chat history and avoid unnecessary duplication of raw conversation text.
38. Player conversation data is retained for 12 months after the last conversation activity unless earlier deletion is triggered through the internal/admin workflow.
39. Derived memory, relationship state, and other conversation-owned artifacts are hard-deleted within 30 days after confirmed conversation or account deletion and do not outlive the owning conversation lifecycle.
40. Audit-minimum safety and operational trace data is retained for 90 days and limited to the minimum metadata and evidence needed for abuse review, policy enforcement, and operational incident investigation.
41. Phase 1 does not expose a Player-facing self-service delete function; deletion is handled through an internal or admin-managed workflow.

## Open Questions
- No open questions are currently blocking the phase 1 requirements baseline.

## Next Step
Reconcile the architecture packet to the new phase 1 policy baseline: hosted-model non-explicit romance only, explicit retention windows, and admin-managed deletion flow.