# Renai Document Sync Tracker

## Purpose
Use this tracker to see which role is aligned to the latest upstream document version and how much of the downstream work has been reconciled.

## Status Legend
| Sync Status | Meaning |
| --- | --- |
| `aligned` | The artifact is synced to the latest upstream version. |
| `needs-sync` | The upstream version changed and this artifact has not been updated yet. |
| `in-progress` | Reconciliation work is underway but not complete. |
| `ready-to-start` | Upstream planning is aligned and execution can begin. |
| `blocked` | Progress cannot continue until an upstream decision is resolved. |

## Status Register
| Role | Scope | Current Version | Upstream Version | Sync Status | Progress | Last Updated | Next Action |
| --- | --- | --- | --- | --- | --- | --- | --- |
| PM | `docs/01_requirements/renai-game-llm-prd.md` | `v1.0` | `n/a` | `aligned` | `100%` | `2026-04-26` | Update the PRD version and change log on the next requirement change. |
| SA | Architecture packet under `docs/02_architecture/` | `v1.0` | `PRD v1.0` | `aligned` | `100%` | `2026-04-26` | Re-open architecture sync only when the PRD version changes or a new architecture artifact is introduced. |
| TL | Planning packet under `docs/03_implementation/` | `v1.0` | `Architecture v1.0` | `aligned` | `100%` | `2026-04-26` | Keep the blueprint, API contracts, backlog, and task artifact aligned to the next architecture revision. |
| Implementor | Execution backlog and task artifact | `v1.0` | `Planning v1.0` | `ready-to-start` | `0%` | `2026-04-26` | Start with `M0` decisions and use the Implementor Execution Tracker agent to keep task statuses and progress updates aligned. |

## Included Artifacts

### PM Scope
- `docs/01_requirements/renai-game-llm-prd.md`

### SA Scope
- `docs/02_architecture/adr-0001-phase1-hosted-model-adult-content-fallback.md`
- `docs/02_architecture/renai-game-llm-mvp-architecture-overview.md`
- `docs/02_architecture/renai-game-llm-mvp-component-diagrams.md`
- `docs/02_architecture/renai-game-llm-mvp-erd.md`
- `docs/02_architecture/renai-game-llm-mvp-hld.md`
- `docs/02_architecture/renai-game-llm-mvp-privacy-retention-architecture.md`
- `docs/02_architecture/renai-game-llm-mvp-security-architecture.md`
- `docs/02_architecture/renai-game-llm-mvp-sequence-flows.md`

### TL Scope
- `docs/03_implementation/renai-game-llm-mvp-api-contract-drafts.md`
- `docs/03_implementation/renai-game-llm-mvp-execution-backlog.md`
- `docs/03_implementation/renai-game-llm-mvp-implementation-blueprint.md`
- `docs/03_implementation/renai-game-llm-mvp-task-artifact.json`

### Implementor Scope
- `docs/03_implementation/renai-game-llm-mvp-execution-backlog.md`
- `docs/03_implementation/renai-game-llm-mvp-task-artifact.json`
- `.github/agents/implementor-execution.agent.md`

## Update Rules
1. When a PRD changes, PM updates the PRD version, appends a `Change Log` row, and marks affected downstream rows as `needs-sync`.
2. When architecture docs are reconciled to the latest PRD, SA updates the architecture row with the new upstream version, progress percentage, and next action.
3. When implementation-planning docs are reconciled to the latest architecture baseline, TL updates the planning row and keeps any task artifact status changes consistent.
4. When implementation work begins, the Implementor updates progress based on the execution backlog and task artifact rather than ad hoc estimates.