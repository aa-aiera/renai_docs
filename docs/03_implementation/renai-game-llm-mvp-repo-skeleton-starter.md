# Renai Game LLM MVP - Repo Skeleton Starter

## Document Control
- Status: Draft Skeleton Starter
- Version: v1.0.0
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.0.0 | Major | Established the repo skeleton starter as a managed implementation support artifact that maps planned scaffold files to their minimal initial contents. | Implementors can create the first-pass repository skeleton without inventing file purpose or starter contents during M1. |

## Upstream Baseline
- Based On: MVP implementation blueprint v1.2.2
- Based On: MVP execution backlog v1.2.2
- Based On: MVP M1 scaffold checklist v1.0.0

## Purpose
This document describes the minimal contents each key scaffold file should start with. It does not provide production-ready implementation. It provides just enough structure that the codebase can compile, run bootstrap tooling, and receive feature work in the correct place.

## Usage Rules
- Start minimal and compilable; do not add speculative feature branches into the skeleton.
- Keep placeholder implementations explicit with TODO markers only where a later backlog item will replace them.
- Prefer empty interfaces or constructors over fake business logic.
- Keep secrets out of all example files.

## Browser Client Starter Files
| File | Minimal Initial Contents |
| --- | --- |
| `apps/web-client/package.json` | package name, `private: true`, scripts for `dev`, `build`, `preview`, `test`, `test:e2e`, and dependencies for React, React DOM, React Router, TanStack Query, Zustand, Vite, Vitest, Testing Library, and Playwright |
| `apps/web-client/tsconfig.json` | strict TypeScript config for Vite React project with path aliases only if they are used immediately |
| `apps/web-client/vite.config.ts` | React plugin registration and default env loading |
| `apps/web-client/vitest.config.ts` | jsdom test environment and setup file registration |
| `apps/web-client/playwright.config.ts` | base URL placeholder and browser test directory definition |
| `apps/web-client/index.html` | Vite root HTML mounting `src/main.tsx` |
| `apps/web-client/.env.example` | `VITE_APP_ENV`, `VITE_API_BASE_URL`, and `VITE_FACEBOOK_APP_ID` placeholders |
| `apps/web-client/src/main.tsx` | React root render with `StrictMode` and app providers |
| `apps/web-client/src/app/App.tsx` | top-level app shell returning the router provider or shared layout wrapper |
| `apps/web-client/src/app/router.tsx` | route definitions for `/`, `/mcs`, `/chat/:mcId`, and `/auth/facebook/callback` |
| `apps/web-client/src/app/providers.tsx` | composition of `QueryClientProvider` and any global state or theme wrapper |
| `apps/web-client/src/app/queryClient.ts` | one exported TanStack Query client instance |
| `apps/web-client/src/routes/home/HomePage.tsx` | placeholder landing screen component |
| `apps/web-client/src/routes/mc-discovery/McDiscoveryPage.tsx` | placeholder MC discovery page component |
| `apps/web-client/src/routes/chat/ChatPage.tsx` | placeholder authenticated chat page component |
| `apps/web-client/src/routes/auth/AuthCallbackPage.tsx` | placeholder auth callback page that consumes query params and triggers auth exchange flow |
| `apps/web-client/src/components/auth/AuthRequiredGate.tsx` | conditional wrapper that renders children or login-required UI |
| `apps/web-client/src/components/chat/DelayedMessageStatus.tsx` | minimal pending-reply indicator component |
| `apps/web-client/src/components/chat/ConversationList.tsx` | minimal conversation summary list component |
| `apps/web-client/src/components/mc/McProfileCard.tsx` | minimal MC profile card component |
| `apps/web-client/src/services/http/apiClient.ts` | shared fetch wrapper with `credentials: include`, JSON parsing, and basic error normalization |
| `apps/web-client/src/services/auth/sessionApi.ts` | session inspection request function calling `/v1/me/session` |
| `apps/web-client/src/services/auth/facebookAuth.ts` | auth exchange request function calling `/v1/auth/facebook/exchange` |
| `apps/web-client/src/services/chat/chatApi.ts` | conversation resolve, load, send, and poll request helpers |
| `apps/web-client/src/state/sessionStore.ts` | Zustand store for authenticated session summary and login-required UI flags |
| `apps/web-client/src/state/conversationStore.ts` | Zustand store for selected conversation-local UI state only |
| `apps/web-client/src/state/mcStore.ts` | Zustand store for current discovery filters or selected MC metadata |
| `apps/web-client/src/styles/tokens.css` | initial design token variables and shared CSS reset or primitives |
| `apps/web-client/src/test/setup.ts` | Testing Library setup import and common test hooks |

## Backend API Starter Files
| File | Minimal Initial Contents |
| --- | --- |
| `services/backend-api/go.mod` | module path plus dependencies for chi, validator, pgx, sqlc runtime, goose, go-redis, amqp091-go, slog helpers, and testify |
| `services/backend-api/.env.example` | placeholders for runtime, cookie, PostgreSQL, Redis, RabbitMQ, Facebook, LLM, retention, and internal admin secret keys |
| `services/backend-api/cmd/api/main.go` | `main()` that loads config, initializes logger and infra clients, builds router, and starts HTTP server with graceful shutdown |
| `services/backend-api/internal/app/app.go` | one app constructor tying config, infra clients, services, and router together |
| `services/backend-api/internal/config/config.go` | config struct definitions and env parsing with validation |
| `services/backend-api/internal/platform/logging/logger.go` | slog logger constructor with environment-aware formatting |
| `services/backend-api/internal/platform/postgres/pool.go` | pgx pool constructor and shutdown helper |
| `services/backend-api/internal/platform/redis/client.go` | go-redis client constructor |
| `services/backend-api/internal/platform/rabbitmq/publisher.go` | RabbitMQ publish abstraction for job dispatch |
| `services/backend-api/internal/platform/session/store.go` | Redis-backed session store interface and opaque session ID operations |
| `services/backend-api/internal/platform/facebook/client.go` | thin Facebook verification client placeholder |
| `services/backend-api/internal/platform/llm/client.go` | thin hosted-provider client placeholder |
| `services/backend-api/internal/http/router/router.go` | central chi router registration with grouped routes and middleware wiring |
| `services/backend-api/internal/http/middleware/origin/origin.go` | origin validation middleware placeholder |
| `services/backend-api/internal/http/middleware/session/session.go` | session lookup middleware placeholder |
| `services/backend-api/internal/http/middleware/requestid/request_id.go` | request ID middleware using context and slog integration |
| `services/backend-api/internal/http/handlers/session/handler.go` | handler struct and `GET /v1/me/session` placeholder |
| `services/backend-api/internal/http/handlers/auth/handler.go` | handler struct and Facebook exchange placeholder |
| `services/backend-api/internal/http/handlers/mc/handler.go` | handler struct and MC list/detail placeholders |
| `services/backend-api/internal/http/handlers/conversation/handler.go` | handler struct and conversation resolve/load placeholders |
| `services/backend-api/internal/http/handlers/message/handler.go` | handler struct and send or poll placeholders |
| `services/backend-api/internal/http/handlers/internal_lifecycle/handler.go` | handler struct and privileged delete placeholder |
| `services/backend-api/internal/modules/identity/service.go` | service struct, constructor, and interface placeholders for auth/session operations |
| `services/backend-api/internal/modules/mc_catalog/service.go` | service struct and MC catalog interface placeholder |
| `services/backend-api/internal/modules/conversations/service.go` | service struct and conversation ownership placeholder methods |
| `services/backend-api/internal/modules/messages/service.go` | service struct and transcript persistence placeholder methods |
| `services/backend-api/internal/modules/chat_orchestration/service.go` | service struct and accepted-message scheduling placeholder method |
| `services/backend-api/internal/modules/memory/service.go` | service struct and prompt-context placeholder methods |
| `services/backend-api/internal/modules/relationship/service.go` | service struct and score-loading placeholder methods |
| `services/backend-api/internal/modules/policy_safety/service.go` | service struct and policy-check placeholder methods |
| `services/backend-api/internal/modules/provider/service.go` | service struct and provider request placeholder methods |
| `services/backend-api/internal/modules/audit/service.go` | service struct and audit emit placeholder methods |
| `services/backend-api/internal/modules/admin_lifecycle/service.go` | service struct and internal delete orchestration placeholder methods |
| `services/backend-api/sqlc.yaml` | sqlc config pointing at `db/queries` and `db/migrations` generated package targets |
| `services/backend-api/db/migrations/0001_initial_schema.sql` | initial schema file with header and placeholder statements for Player, MC, conversation, message, memory, relationship, reply schedule, and audit tables |
| `services/backend-api/db/queries/conversations.sql` | query placeholders for load or create conversation operations |
| `services/backend-api/db/queries/messages.sql` | query placeholders for message insert and transcript reads |
| `services/backend-api/db/queries/memory.sql` | query placeholders for short-term and long-term memory reads or writes |
| `services/backend-api/db/queries/relationship.sql` | query placeholders for relationship state reads or updates |
| `services/backend-api/db/queries/audit.sql` | query placeholders for metadata-first audit inserts |
| `services/backend-api/tests/integration/` | test package placeholder with one smoke test file once implementation starts |

## Worker Starter Files
| File | Minimal Initial Contents |
| --- | --- |
| `services/worker/go.mod` | module path and dependencies aligned to RabbitMQ, Redis, pgx, validator if reused, slog, and testify |
| `services/worker/.env.example` | placeholders for runtime, PostgreSQL, Redis, RabbitMQ, LLM provider, and retention keys |
| `services/worker/cmd/worker/main.go` | `main()` that loads config, initializes clients, registers consumers, and blocks on graceful shutdown |
| `services/worker/internal/app/worker.go` | worker app constructor and consumer registration surface |
| `services/worker/internal/config/config.go` | config struct and env parsing for worker-only needs |
| `services/worker/internal/platform/logging/logger.go` | slog logger constructor |
| `services/worker/internal/platform/postgres/pool.go` | pgx pool constructor |
| `services/worker/internal/platform/redis/client.go` | go-redis client constructor |
| `services/worker/internal/platform/rabbitmq/consumer.go` | RabbitMQ consumer setup abstraction |
| `services/worker/internal/platform/llm/client.go` | hosted-provider client placeholder |
| `services/worker/internal/jobs/execute_delayed_reply/handler.go` | handler struct and queue message decode placeholder |
| `services/worker/internal/jobs/update_short_term_context/handler.go` | handler struct and placeholder processing method |
| `services/worker/internal/jobs/update_long_term_memory/handler.go` | handler struct and placeholder processing method |
| `services/worker/internal/jobs/update_relationship_state/handler.go` | handler struct and placeholder processing method |
| `services/worker/internal/jobs/purge_conversation/handler.go` | handler struct and placeholder processing method |
| `services/worker/internal/jobs/emit_audit_events/handler.go` | handler struct and placeholder processing method |
| `services/worker/tests/integration/` | test package placeholder for future queue and database smoke coverage |

## Infrastructure Starter Files
| File | Minimal Initial Contents |
| --- | --- |
| `infra/phase1/env-template/web-client.env.example` | public browser env keys only |
| `infra/phase1/env-template/backend-api.env.example` | backend env keys with empty secret placeholders and default retention values |
| `infra/phase1/env-template/worker.env.example` | worker env keys with empty secret placeholders and default retention values |
| `infra/phase1/env-template/shared.infra.env.example` | local PostgreSQL, Redis, and RabbitMQ bootstrap keys only |
| `infra/phase1/docker-compose/docker-compose.yml` | service definitions for postgres, redis, rabbitmq, backend-api, worker, and web-client placeholders |
| `infra/phase1/app-config/backend-api.config.example.yaml` | optional non-secret runtime config example if the Go service uses YAML overlays |
| `infra/phase1/app-config/worker.config.example.yaml` | optional worker runtime config example |
| `infra/phase1/queue-config/rabbitmq.definitions.example.json` | exchange, queue, and binding placeholders for delayed reply and background job queues |
| `infra/phase1/db-config/postgres.init.example.sql` | optional bootstrap SQL for local roles, extensions, or database defaults |

## Naming Guardrails
- React route folders use kebab-case and page component files use PascalCase.
- Go package directories stay lowercase with underscores only where the selected scaffold already requires them.
- Queue job directories stay in snake_case to mirror queue names.
- Example env and config files always end in `.example` and never contain secrets.

## Open Questions
No open questions are currently blocking the repo skeleton starter.

## Recommendation Summary
Recommendation:
Use this document when creating the first pass of the repository. Keep each file minimal, compilable, and correctly placed, then let the backlog drive when placeholders are replaced with real behavior.
