# Renai Game LLM MVP - M1 Scaffold Checklist

## Document Control
- Status: Draft Scaffold Checklist
- Version: v1.0.0
- Last Updated: 2026-04-26
- Owner: TL

## Change Log
| Date | Version | Change Type | Summary | Downstream Impact |
| --- | --- | --- | --- | --- |
| 2026-04-26 | v1.0.0 | Major | Established the M1 scaffold checklist as a managed implementation support artifact with strict file creation order, dependency notes, and acceptance checks for the selected React and Go stack. | Implementors can start repository scaffolding in a deterministic order without reopening stack, session, or env-template decisions. |

## Upstream Baseline
- Based On: MVP architecture packet v1.1.0
- Based On: MVP implementation blueprint v1.2.2
- Based On: MVP execution backlog v1.2.2

## Purpose
This checklist converts the strict implementation blueprint into an execution-order scaffold guide for M0 and M1. It is not a backlog replacement. Its job is to tell the implementor what to create first, what each step depends on, and how to know when the scaffold is ready for the next step.

## Entry Criteria
Before starting this checklist:
- the selected stack baseline is fixed to React + TypeScript + Vite, Go, PostgreSQL, Redis, and RabbitMQ
- session transport is fixed to a secure `HttpOnly` opaque cookie backed by Redis
- secret distribution is fixed to managed-secret injection outside the repository
- the implementor is not changing product or architecture scope while scaffolding

## Execution Rules
- Create files in the order below unless a concrete repository constraint forces a small local swap.
- Do not add production business logic while scaffold work is still incomplete.
- Keep env example files secret-free.
- Keep browser HTTP transport centralized in one shared client with `credentials: include`.
- Keep Go entrypoints thin and push business behavior into `internal/modules/`.

## Ordered Checklist
| Step | Objective | Create Or Update | Depends On | Done When |
| --- | --- | --- | --- | --- |
| `SCF-001` | Create top-level repo structure | `apps/web-client/`, `services/backend-api/`, `services/worker/`, `infra/phase1/`, `tests/` | none | top-level directories exist and match the blueprint naming baseline |
| `SCF-002` | Bootstrap browser package and build tooling | `apps/web-client/package.json`, `tsconfig.json`, `vite.config.ts`, `vitest.config.ts`, `playwright.config.ts`, `index.html`, `.env.example` | `SCF-001` | browser workspace can install dependencies and exposes the selected scripts and env placeholders |
| `SCF-003` | Bootstrap browser app shell | `apps/web-client/src/main.tsx`, `src/app/App.tsx`, `src/app/router.tsx`, `src/app/providers.tsx`, `src/app/queryClient.ts` | `SCF-002` | browser shell can mount routes and shared providers without feature logic |
| `SCF-004` | Create browser route pages and components | `src/routes/home/HomePage.tsx`, `src/routes/mc-discovery/McDiscoveryPage.tsx`, `src/routes/chat/ChatPage.tsx`, `src/routes/auth/AuthCallbackPage.tsx`, `src/components/auth/AuthRequiredGate.tsx`, `src/components/chat/DelayedMessageStatus.tsx`, `src/components/chat/ConversationList.tsx`, `src/components/mc/McProfileCard.tsx` | `SCF-003` | the route tree is present and each route resolves to a named page component |
| `SCF-005` | Create browser transport and stores | `src/services/http/apiClient.ts`, `src/services/auth/sessionApi.ts`, `src/services/auth/facebookAuth.ts`, `src/services/chat/chatApi.ts`, `src/state/sessionStore.ts`, `src/state/conversationStore.ts`, `src/state/mcStore.ts`, `src/styles/tokens.css`, `src/test/setup.ts` | `SCF-003` | API access, local stores, and test setup are present with the selected tool conventions |
| `SCF-006` | Bootstrap backend API module and entrypoint | `services/backend-api/go.mod`, `go.sum`, `.env.example`, `cmd/api/main.go`, `internal/app/app.go` | `SCF-001` | API service module exists and can wire config, logger, infrastructure clients, and router entrypoints |
| `SCF-007` | Create backend config and infrastructure adapters | `internal/config/config.go`, `internal/platform/logging/logger.go`, `internal/platform/postgres/pool.go`, `internal/platform/redis/client.go`, `internal/platform/rabbitmq/publisher.go`, `internal/platform/session/store.go`, `internal/platform/facebook/client.go`, `internal/platform/llm/client.go` | `SCF-006` | API service can load env config and declare infrastructure client construction points |
| `SCF-008` | Create backend router, middleware, and handler skeletons | `internal/http/router/router.go`, `internal/http/middleware/origin/origin.go`, `internal/http/middleware/session/session.go`, `internal/http/middleware/requestid/request_id.go`, `internal/http/handlers/session/handler.go`, `internal/http/handlers/auth/handler.go`, `internal/http/handlers/mc/handler.go`, `internal/http/handlers/conversation/handler.go`, `internal/http/handlers/message/handler.go`, `internal/http/handlers/internal_lifecycle/handler.go` | `SCF-007` | all planned HTTP surfaces have a route and handler home with no orphan endpoint decisions |
| `SCF-009` | Create backend module service skeletons | `internal/modules/identity/service.go`, `internal/modules/mc_catalog/service.go`, `internal/modules/conversations/service.go`, `internal/modules/messages/service.go`, `internal/modules/chat_orchestration/service.go`, `internal/modules/memory/service.go`, `internal/modules/relationship/service.go`, `internal/modules/policy_safety/service.go`, `internal/modules/provider/service.go`, `internal/modules/audit/service.go`, `internal/modules/admin_lifecycle/service.go` | `SCF-007` | each module exists as a business-logic home with constructor and interface placeholders |
| `SCF-010` | Create database scaffold and query generation inputs | `services/backend-api/sqlc.yaml`, `db/migrations/0001_initial_schema.sql`, `db/queries/conversations.sql`, `db/queries/messages.sql`, `db/queries/memory.sql`, `db/queries/relationship.sql`, `db/queries/audit.sql` | `SCF-007` | migration and query directories exist and are ready for schema-first implementation work |
| `SCF-011` | Bootstrap worker module and job skeletons | `services/worker/go.mod`, `go.sum`, `.env.example`, `cmd/worker/main.go`, `internal/app/worker.go`, `internal/config/config.go`, `internal/platform/logging/logger.go`, `internal/platform/postgres/pool.go`, `internal/platform/redis/client.go`, `internal/platform/rabbitmq/consumer.go`, `internal/platform/llm/client.go`, `internal/jobs/execute_delayed_reply/handler.go`, `internal/jobs/update_short_term_context/handler.go`, `internal/jobs/update_long_term_memory/handler.go`, `internal/jobs/update_relationship_state/handler.go`, `internal/jobs/purge_conversation/handler.go`, `internal/jobs/emit_audit_events/handler.go` | `SCF-001` | worker module exists and every planned queue job has an owning handler file |
| `SCF-012` | Create integration-test and infra examples | `services/backend-api/tests/integration/`, `services/worker/tests/integration/`, `infra/phase1/env-template/web-client.env.example`, `infra/phase1/env-template/backend-api.env.example`, `infra/phase1/env-template/worker.env.example`, `infra/phase1/env-template/shared.infra.env.example`, `infra/phase1/docker-compose/docker-compose.yml`, `infra/phase1/app-config/backend-api.config.example.yaml`, `infra/phase1/app-config/worker.config.example.yaml`, `infra/phase1/queue-config/rabbitmq.definitions.example.json`, `infra/phase1/db-config/postgres.init.example.sql` | `SCF-002`, `SCF-006`, `SCF-011` | local bootstrap files and test directories exist and reflect the selected env-key baseline |

## Dependency Notes
- `SCF-002` to `SCF-005` are browser-only and can proceed in parallel with parts of `SCF-006` to `SCF-010` once `SCF-001` is done.
- `SCF-010` should not be skipped because PostgreSQL query generation and migration layout are part of the selected stack baseline.
- `SCF-011` should mirror naming from the API service to reduce operator confusion later.
- `SCF-012` is not optional; env examples and local infra bootstrap are part of the scaffold contract.

## Acceptance Sweep
Use this final sweep before marking scaffold work complete:
- each planned deployable service has its own entrypoint
- browser HTTP transport is centralized
- cookie-session config exists only in backend and env-template files
- no secrets are committed into example files
- PostgreSQL, Redis, and RabbitMQ config keys are present in env templates
- queue job handlers exist for delayed reply, memory, relationship, purge, and audit work
- route page names and Go package names follow the agreed naming rules

## Open Questions
No open questions are currently blocking the scaffold checklist.

## Recommendation Summary
Recommendation:
Treat this checklist as the execution order for M1. Finish scaffold creation before writing feature logic, and use the exact file paths here as the baseline that backlog tasks and the implementation blueprint now expect.
