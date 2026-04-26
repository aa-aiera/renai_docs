# Renai Game Using LLM - Draft PRD

## Document Control
- Status: Draft Phase 1 MVP PRD
- Version: v1.0
- Last Updated: 2026-04-26
- Owner: PM

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.0 | Major | Established the current phase 1 MVP PRD as the managed requirements baseline. | Architecture and implementation planning should track against PRD v1.0 until a newer requirements version is published. |

## Request Summary
Create a renai-game-style experience (เกมส์จีบสาว / romance simulation) using an LLM.

This product should feel closer to a dating app such as Tinder than to a branching visual novel. Players browse or select AI characters and chat with them directly. The core experience is open-ended LLM conversation, not an if/else scenario tree.

## Confirmed Requirements
- The product should be in the style of a renai game.
- The product should use an LLM as a core capability.
- The experience should be dating-app-like rather than a scripted route-selection game.
- Phase 1 should target both web and mobile.
- The first milestone should be an MVP.
- The MVP should launch with the first 3 main characters.
- The MVP should be delivered as a responsive browser experience.
- Phase 1 should use a browser-based responsive UI rather than separate native apps.
- Returning-player identity in phase 1 should start with Facebook login.
- Future identity should support linked multi-provider accounts such as Facebook, LINE, and email.
- Phase 1 should allow non-logged-in users to try chatting before login.
- Guest trial users should be limited to 10 user input chat messages.
- Login should be required after the guest trial limit to continue chatting as a persistent player.
- Guest trial chat history, memory, and relationship state should reset at login and should not transfer into the Facebook-linked account.
- Guest trial data should be disposable and eligible for expiry and purge after the guest lifecycle ends.
- Mobile should use swipe-style character browsing.
- Web should use a character-list browsing experience.
- Users should be able to select an AI character and chat with that character directly.
- The conversation model should be open-ended, not driven by fixed if/else scenarios.
- Replies should not always arrive immediately; response timing should feel more human.
- Normal non-delayed responses should simulate a simple human pacing model.
- For normal non-delayed replies, the system should simulate 1 to 3 seconds of reading time and 1 to 3 seconds of response time.
- If a character is less interested in a user, the response delay may be longer, but should not exceed 5 minutes.
- Delayed replies should be presented as simple message delay.
- Each character should have a defined persona, talking style, focus areas, and personal interests.
- Each character's knowledge should be bounded by their background so they do not feel unrealistically omniscient.
- The system should store both short-term memory and long-term memory for each user-character pair.
- Short-term and long-term memory should be saved as text so it can be reused as LLM context.
- If user A continues chatting with character B over time, character B should know user A better based on prior interactions.
- Phase 1 should not include a shared-world memory model.
- Phase 1 should not include cross-character memory rules or character-to-character memory sharing.
- The game should track relationship status metrics such as attraction, friendship, and trust.
- The product should support both friendship-style and romance-style relationship progression.
- Conversation tone should change based on character profile and the relationship status with each player.
- Derived short-term context, long-term memory, and relationship state should follow the lifecycle of the owning conversation and should not persist independently after purge.
- If friendship is high, characters may shift into deeper platonic conversation.
- If romantic relationship state is high enough, characters may speak more like a lover.
- Content boundaries should depend on the character definition and the relationship with the player rather than one global rule.
- Safety restrictions are critical and must be enforced across all characters.
- Characters must not make important life decisions on behalf of players.
- Characters must not persuade or encourage players to do illegal actions.
- Characters must not position themselves as a general-purpose problem solver for serious real-world issues.
- In phase 1, the product language should be Thai.
- Sexual content should be gated in phase 1 by a simple self-attested 18+ confirmation dialog.
- Players under 18 should not receive sexual content.
- Adult sexual content intent is limited to players who confirm they are 18+ and interact with adult characters.
- In phase 2, each character should have a native language, and some players may have a second-language proficiency level such as moderate, good, or fluent.
- Phase 1 players should not see relationship scores.
- Premium score visibility should be deferred to the next phase.
- When premium score visibility is introduced, approved visible premium metrics should unlock immediately.
- Some scores should remain secret even in premium phases.
- Phase 1 should use a public hosted model such as Gemini.
- Phase 1 AI token cost should be capped at $1000.
- The $1000 AI token budget cap should apply to the MVP test cycle.
- Phase 1 backend should support future scale usage, even if the initial deployment is simpler.
- In phase 1, the full system should be able to run on a medium-sized instance, potentially using Docker deployment.
- Phase 2 should support replacing the hosted model with a local LLM.
- Audit and safety trace data should be minimized and should not become a duplicate player-visible transcript store.

## Assumptions
- The initial chat experience is one user interacting with one AI character at a time.
- Detailed duration and traffic assumptions for budget-cap analysis are deferred in the current MVP planning pass.

## Product Goals
- Deliver an MVP in a responsive browser format so players on desktop and mobile can access the same first release.
- Deliver a first playable loop where a user selects an AI character and starts a natural one-on-one conversation.
- Create a sense of emotional continuity by having characters remember the user across repeated chats.
- Make replies feel socially believable by simulating non-instant response timing.
- Preserve believability by keeping each character's knowledge, tone, and interests aligned to their persona.
- Support more than romance-only play by allowing believable friendship progression as well.
- Match interaction patterns to platform expectations: swipe-based discovery on mobile and list-based browsing on web.
- Avoid brittle script-branch maintenance by relying on LLM-driven dialogue instead of if/else scenarios.
- Keep the product architecture and requirements flexible enough to migrate from a hosted public model to a local LLM later.

## Target Users
- Thai-speaking players in phase 1 who are interested in romance-simulation, friendship-simulation, or AI companion chat experiences.
- Early adopters who value conversational freedom more than fixed story routes.

## Problem Statement
Traditional renai or dating-sim experiences often depend on prewritten dialogue trees and branching logic. That structure limits conversational freedom and makes it hard for characters to feel personally aware of the player. This product aims to provide a more natural romance-chat experience where AI characters can talk openly and remember the player over time.

## Scope
### In Scope
- Dating-app-style character discovery and selection.
- A responsive browser-based experience for phase 1 that works across desktop and mobile browsers.
- MVP scope with the first 3 main characters.
- Swipe-style character browsing on mobile.
- Character-list browsing on web.
- One-on-one chat between a user and a selected AI character.
- Character personas that shape tone, style, and behavior in conversation.
- Character-specific response timing behavior.
- Character-specific knowledge boundaries based on age, background, and profile.
- Dynamic conversation style that can remain platonic or become more romantic depending on the relationship state.
- Short-term memory for recent dialogue context.
- Long-term memory for persistent facts, preferences, and relationship history.
- Text-based memory records and summaries used as context for the LLM.
- Relationship state metrics such as attraction, friendship, and trust.
- Tier-based visibility rules for relationship metrics.
- Thai language support in phase 1.
- Expanded language behavior in phase 2, including character native language and optional player second-language proficiency.
- Re-entry into prior conversations with continuity for the same user-character pair.
- Phase 1 launch using a hosted public LLM such as Gemini.
- Phase 2 migration path to a local LLM without changing the core player workflow.

### Out of Scope
- Branching if/else scenario trees as the primary interaction model.
- A traditional visual-novel route engine for the initial product.
- Local-LLM-only deployment in phase 1.
- Separate native mobile apps for the MVP.
- Shared memory across all characters by default.
- Shared-world memory models.
- Cross-character memory rules or dynamic character-to-character memory sharing.
- A single global romance-only content mode for all characters.

## Functional Requirements
1. Users shall be able to browse or select from multiple AI characters.
2. The first milestone shall be an MVP delivered through a responsive browser experience.
3. The MVP shall launch with 3 main characters.
4. Phase 1 shall support both desktop-browser and mobile-browser access through the same responsive web product.
5. Phase 1 returning-player identity shall use Facebook login so chat continuity, memory, and relationship state can persist across sessions.
6. Future phases shall support linked multi-provider account identity, including at minimum Facebook, LINE, and email.
7. On mobile, users shall browse characters primarily through a swipe-style interface.
8. On web, users shall browse characters through a character-list interface.
9. Users shall be able to open a dedicated one-on-one chat with a selected character.
10. Each character shall have a distinct persona so replies feel character-specific rather than generic.
11. Each character definition shall include at minimum persona, talking style, focus areas, interests, and background attributes relevant to conversation behavior.
12. The system shall use character background constraints so a character's knowledge feels appropriate to that character rather than universally expert.
13. For example, a high-school character should not speak like an expert in adult life or deep historical analysis unless their profile explicitly justifies it.
14. The system shall support human-like response timing instead of always returning replies immediately.
15. For normal non-delayed replies, the system shall simulate simple human pacing with 1 to 3 seconds of reading time and 1 to 3 seconds of response time.
16. Response timing may vary by character state, including lower interest in the user, but the delayed reply window shall not exceed 5 minutes.
17. Delayed replies shall be presented as simple message delay rather than a complex simulated activity system.
18. The main interaction shall be free-text conversation powered by an LLM instead of pre-scripted if/else choices.
19. The system shall maintain short-term memory during active conversation so the character can respond coherently to recent messages.
20. The system shall maintain long-term memory across sessions so the same character can remember important facts about the same user later.
21. Short-term and long-term memory shall be stored as text artifacts suitable for reuse in LLM context assembly.
22. Long-term memory shall be scoped to the user-character pair only.
23. Phase 1 shall not include a shared-world memory model.
24. Phase 1 shall not include cross-character memory rules or character-to-character memory sharing.
25. Returning to the same character shall restore enough prior context for the relationship to feel continuous.
26. The product shall compute and store relationship-state metrics for each user-character pair, including at minimum attraction, friendship, and trust.
27. Relationship-state metrics shall be updated over time based on conversation content and interaction history.
28. The relationship system shall support both platonic and romantic progression paths.
29. Character conversation style shall adapt to relationship state. High friendship may unlock deeper platonic discussion, while higher romantic state may unlock more lover-like conversation where appropriate to the character.
30. Score visibility shall be tier-based.
31. Default players shall not see relationship scores by default.
32. Premium players may see only the subset of scores designated as visible.
33. Some scores shall remain hidden from all player tiers.
34. Phase 1 product-facing experience shall be Thai-first.
35. Phase 1 shall enforce age-gated sexual content through a simple self-attested 18+ confirmation dialog.
36. Players under 18 shall not receive sexual content.
37. Adult sexual content intent shall apply only to players who confirm they are 18+ and interact with adult characters.
38. Phase 2 shall support characters with native languages and optional player second-language proficiency states such as moderate, good, or fluent.
39. All characters shall follow behavioral safety restrictions regardless of persona, relationship level, or player tier.
40. Characters shall not make important life decisions for players, including major decisions about health, finance, education, family, safety, or long-term life direction.
41. Characters shall not persuade, coach, or encourage users to perform illegal acts.
42. Characters shall not present themselves as a general-purpose solver for serious real-world personal problems.
43. When users ask for dangerous, illegal, or high-stakes life guidance, the character response style shall stay in-character while refusing the unsafe role.
44. Phase 1 infrastructure shall be able to run the full system on a medium-sized instance, potentially through Docker deployment.
45. Phase 1 architecture shall not block future scaling to higher usage levels.
46. Phase 1 AI token cost shall be managed to stay within the $1000 budget cap for the MVP test cycle.
47. The product shall support a model-provider layer or equivalent product capability so phase 1 can use Gemini or another hosted model and phase 2 can switch to a local LLM.
48. Phase 1 shall allow non-logged-in guest users to start chatting with characters before login.
49. Guest users shall be limited to 10 user input chat messages.
50. After the guest limit is reached, login shall be required to continue chatting as a persistent player.
51. Guest trial chat history, memory, and relationship state shall reset at login and shall not transfer into the Facebook-linked account.
52. Guest session data shall be disposable and shall become eligible for expiry and purge after the guest lifecycle ends.
53. Derived short-term context, long-term memory, and relationship state shall follow the lifecycle of the owning conversation and shall not remain after that conversation is purged.
54. Conversation-owned data shall support lifecycle states so soft-deleted or purge-pending conversations are inaccessible and cannot accept new replies.
55. Audit and safety trace data shall be minimized and retained separately from player-visible conversation history.

## Relationship State Metrics
- The system should maintain relationship scores per user-character pair.
- Confirmed baseline metrics are attraction, friendship, and trust.
- These metrics should represent relationship progress and influence future conversation behavior.
- Phase 1 players should not see these scores.
- Premium-visible score display is deferred to a later phase.
- When premium score display is introduced, approved visible metrics unlock immediately for premium players.
- Some scores should remain secret to all players.
- Friendship progression and romantic progression should be handled as distinct but related behavioral outcomes.

## Recommended Metric Model
Recommendation: use a 0-100 score scale for all relationship and behavior metrics in phase 1.

| Metric | Purpose | Recommended Visibility | Recommended MVP Use |
| --- | --- | --- | --- |
| Friendship | Measures platonic closeness and ease | Future premium-visible candidate | Gates deeper friendly talk and long-term bond progression |
| Trust | Measures reliability, emotional safety, and honesty | Future premium-visible candidate | Gates vulnerable sharing, serious talk, and forgiveness after conflict |
| Attraction | Measures romantic or physical pull | Future premium-visible candidate | Gates flirt tone and romance escalation |
| Comfort | Measures how safe and relaxed the character feels | Future premium-visible candidate | Gates pacing, topic depth, and resistance to pressure |
| Interest / Reply Priority | Measures current eagerness to reply | Secret | Influences response delay and effort level |
| Emotional Openness | Measures willingness to share personal thoughts | Secret | Gates deeper life talk and vulnerability |
| Boundary Safety | Measures whether the player is respecting limits | Secret | Prevents escalation when the player is too pushy or invasive |
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
- Interest / Reply Priority decreases when the user becomes repetitive, spammy, demanding, or conflict-heavy.
- Emotional Openness increases mainly after friendship and trust rise together.
- Boundary Safety decreases when the player ignores a soft no, keeps pressing sensitive topics, or pushes romantic or sexual tone too early.
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
- The system should preserve character consistency across long conversations and repeat sessions.
- The system should preserve knowledge boundaries so characters do not drift into implausible expertise.
- Safety guardrails should remain stable even when character tone, relationship level, or prompt context changes.
- The phase 1 experience should be fully usable in Thai.
- Phase 1 operational cost management should prioritize AI token spend and keep it within the $1000 budget cap for the MVP test cycle.
- Phase 1 deployment should be feasible on a medium-sized instance, potentially containerized with Docker.
- Phase 1 architecture should allow later scaling beyond a single medium-sized instance.
- The product should support provider portability so changing from a hosted model to a local model does not require redefining the player experience.
- User conversation history and memory data should be handled with clear privacy and retention rules.
- Guest-owned data should be intentionally disposable and eligible for cleanup after guest expiry.
- Derived memory and relationship artifacts should be deletable through ownership cascade rather than retained independently.
- Audit data should be minimized so it does not duplicate full chat history unless operationally required.

## Content, Tone, and Safety Constraints
- The core tone is romance-simulation / dating-chat.
- The product should not be limited to romance-only outcomes; friendship-style progression is also a core supported mode.
- Content tone should vary by character definition and by the relationship state with each player.
- A character with high friendship may shift toward deeper talk without necessarily becoming romantic.
- A character with stronger romantic progression may use more lover-like tone where that behavior fits the character profile.
- Sexual content must be gated by verified player age.
- Phase 1 age gating uses a simple self-attested 18+ confirmation dialog.
- Players under 18 must not receive sexual content.
- Adult sexual content intent applies only to players who confirm they are 18+ and interact with adult characters.
- Characters must not make important life decisions for players.
- Characters must not convince or pressure players to do illegal things.
- Characters must not behave as a serious real-world problem solver.
- If a user asks for help in a dangerous, illegal, or high-stakes domain, the character should stay in character while refusing to take on that role.
- These restrictions apply even when the user has a strong relationship with the character.
- Exact safety ceilings and disallowed content categories are still not fully specified beyond the restrictions above.
- Safety controls will be required to enforce the chosen content policy and prevent unsafe or disallowed model output.

## Draft MVP Character Roster
Recommendation: keep all 3 MVP characters as adults to simplify age-gated relationship handling in phase 1.

| Character | Age | Background | Persona and Talking Style | Focus and Interests | Relationship Boundary |
| --- | --- | --- | --- | --- | --- |
| Aira | 22 | Final-year communication design student and part-time cafe barista in Bangkok | Warm, witty, observant, casual Thai tone, playful teasing, light and expressive in chat | Art, music, campus life, coffee, late-night creative work | Starts friendly and playful; romance should require strong trust and comfort; rejects aggressive sexual escalation |
| Lyra | 26 | Elementary school teacher who volunteers on weekends and lives a grounded routine | Gentle, patient, thoughtful, calm pacing, emotionally attentive, low-drama | Books, kids, family, quiet travel, food, daily life reflection | Strong friendship route; slow-burn romance only after consistent trust; reacts negatively to manipulative or pushy behavior |
| Nova | 29 | Marketing strategist in a fast-paced city job with a confident social life | Sharp, confident, teasing, direct, stylish, socially skilled, higher-energy banter | Career, food, fitness, nightlife, urban life, ambition | Can show attraction earlier than the others, but true intimacy still requires trust; disengages when the player becomes controlling or dishonest |

## Release Phasing
| Phase | Goal | Model Strategy |
| --- | --- | --- |
| Phase 1 | Deliver an MVP with 3 main characters in a responsive browser experience and validate memory-based chat loop in Thai | Public hosted model such as Gemini |
| Phase 2 | Improve control, cost profile, language depth, or deployment ownership | Replace hosted model with local LLM |

## First Milestone
- Milestone type: MVP
- Delivery format: Responsive browser application
- Initial character roster: Aira, Lyra, and Nova
- Returning-player identity: Facebook login
- Guest trial access: Non-logged-in users can send up to 10 user input chat messages before login is required
- Guest trial carryover: Guest chat history, memory, and relationship state reset at login
- Device support: Desktop browser and mobile browser
- Backend expectation: Entire phase 1 system can run on a medium-sized instance, potentially using Docker
- Cost constraint: Prioritize keeping AI token cost within a $1000 budget cap for the MVP test cycle
- Normal non-delayed response pacing: Simulate 1 to 3 seconds of reading time and 1 to 3 seconds of response time

## Risks
- LLM output consistency may degrade over time if character personas and memory policies are not clearly defined.
- Memory quality may be poor if the system stores too much irrelevant information or fails to retain important user facts.
- Characters may feel fake if reply timing is too mechanical, too random, or too slow.
- Characters may feel inconsistent if the model ignores persona-specific knowledge boundaries.
- Relationship scores may feel arbitrary if their update rules are not explainable and stable.
- Safety failures may be more likely if the model confuses emotional companionship with permission to give high-stakes guidance.
- Hosted public model policy in phase 1 may conflict with the intended adult-content policy for verified 18+ users.
- Migrating from a hosted model to a local LLM may change tone, quality, latency, or operating cost.
- Safety and content boundaries need to be defined early, especially for romance mechanics, age gating, and moderation.
- Cost and latency may materially affect moment-to-moment play if the game relies on frequent long-form model calls.

## Success Metrics
- Repeat chat sessions per user with the same character.
- Percentage of returning sessions where the character correctly recalls stored user facts.
- Average conversation length per session.
- Percentage of conversations where reply timing feels believable to users.
- Stability of attraction, friendship, and trust score progression across repeated sessions.
- Retention of users who return to continue a prior relationship thread.
- Ability to operate the phase 1 MVP while keeping AI token cost within the $1000 budget cap for the MVP test cycle.

## User Stories
- As a player, I want to browse AI characters so I can pick one that matches my preference.
- As a mobile player, I want to swipe through characters so discovery feels natural on my phone.
- As a web player, I want to view a character list so discovery feels efficient in the browser.
- As a new player, I want to try chatting before login so I can test the experience before connecting my account.
- As a player, I want to chat freely with a selected character so the experience feels natural instead of menu-driven.
- As a returning player, I want the same character to remember important things about me so the relationship feels continuous.
- As a player, I want reply timing to feel human so the chat feels more like talking to a real person.
- As a player, I want each character to speak and think within their own background so they feel believable.
- As a player, I want my relationship with each character to progress through attraction, friendship, and trust over time.
- As a player, I want some characters to become close friends even if the relationship does not become romantic.
- As a future premium player, I want to see some relationship progress indicators while still keeping hidden mechanics secret.
- As a product team, I want the first release to focus on 3 main characters so the MVP is small enough to validate quickly.
- As a product team, I want to launch on a hosted public model first so we can validate the game before moving to a local LLM.

## Acceptance Criteria
1. The first release is an MVP delivered as a responsive browser application.
2. The MVP launches with 3 main characters.
3. Phase 1 supports both desktop-browser and mobile-browser usage.
4. On mobile, users can browse characters through a swipe-style interaction.
5. On web, users can browse characters through a character-list interaction.
6. A user can select an AI character and enter a dedicated chat screen for that character.
7. The user can send free-text messages and receive LLM-generated replies that reflect the selected character persona, talking style, focus, and interests.
8. For normal non-delayed replies, the system simulates 1 to 3 seconds of reading time and 1 to 3 seconds of response time.
9. Replies are not always immediate and can be delayed to simulate human behavior.
10. In low-interest scenarios, delayed replies do not exceed 5 minutes.
11. Delayed replies are presented as simple message delay.
12. During an active conversation, the character can reference recent messages with coherent short-term context.
13. After a user returns to the same character in a later session, the character can recall at least some previously stored facts or interaction history about that user.
14. Short-term and long-term memory are persisted as text and can be reused in prompt context generation.
15. Memory for user A with character B remains separate from memories involving other characters unless a future feature explicitly changes that rule.
16. Character replies remain broadly consistent with the character's knowledge boundary and background.
17. The system stores and updates attraction, friendship, and trust scores for each user-character pair.
18. Phase 1 players do not see relationship scores.
19. Premium-visible score display is not part of the phase 1 MVP.
20. Some scores remain hidden from all players.
21. Phase 1 user-facing experience is available in Thai.
22. Phase 1 uses a simple 18+ confirmation dialog to gate adult sexual content intent.
23. The relationship system supports both friendship-style and romance-style progression.
24. Characters do not make important life decisions on behalf of users.
25. Characters do not encourage or coach illegal behavior.
26. Characters do not present themselves as a serious real-world problem solver.
27. When users request dangerous, illegal, or high-stakes guidance, the character refuses that role while remaining consistent with the product's tone and persona rules.
28. Phase 1 requirements can be satisfied using a hosted public model such as Gemini.
29. The full phase 1 system can run on a medium-sized instance, potentially via Docker deployment.
30. Phase 1 architecture preserves a path to future scaling.
31. Phase 1 AI token cost is managed within the $1000 budget cap for the MVP test cycle.
32. Phase 2 can swap in a local LLM while preserving the same core product flow of selecting a character and chatting.
33. A non-logged-in user can start a guest chat trial with a character.
34. Guest trial users are limited to 10 user input chat messages.
35. After the guest trial limit is reached, login is required to continue chatting.
36. When a guest user logs in, guest chat history, memory, and relationship state reset instead of transferring into the Facebook-linked account.
37. Guest-owned chat data becomes eligible for expiry and purge after the guest lifecycle ends and does not appear as authenticated continuity.
38. Short-term context, long-term memory, and relationship state are removed or made inaccessible when the owning conversation is purged.
39. Conversations marked deleted or purge-pending do not accept new user messages or delayed replies.
40. Audit and safety traces are stored separately from player-visible chat history and avoid unnecessary duplication of raw conversation text.

## Open Questions
1. Can the phase 1 hosted model and provider policy support the intended adult-content policy for players who confirm they are 18+, or must adult sexual content wait for a later phase or local model?
2. What concrete retention durations should apply to guest-limited, account-durable, and audit-minimum data classes in phase 1?
3. Should phase 1 expose any user-facing delete capability, or should delete remain an internal or admin-managed lifecycle path until later?

## Next Step
Confirm the phase 1 hosted-model content posture and finalize retention-duration and delete-exposure policy values so the architecture packet can move from lifecycle classes to implementation-ready policy settings.