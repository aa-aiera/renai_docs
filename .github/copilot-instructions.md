# Document Governance Rules

This repository is a document pipeline: product requirements drive architecture, architecture drives implementation planning, and implementation planning drives execution.

When you create or update documents in this workspace, preserve change traceability instead of making silent edits.

## Versioning Standard

- Use document versions in `v<release_version>.<major_change>.<minor_change>` format.
- `release_version` identifies the active release line or release baseline for the document set.
- `major_change` increments when a change invalidates or materially reshapes the current system direction within the same release line. Examples: scope changes, workflow changes, new or removed capabilities, safety or policy posture changes, data-model-impacting changes, or any requirement change that forces downstream redesign.
- `minor_change` increments for compatible refinements within the same release line and major-change baseline. Examples: clarified wording, added acceptance criteria, scoped details, non-breaking constraints, terminology cleanup, or decisions that do not force downstream redesign.
- When `release_version` increments, reset `major_change` and `minor_change` to `0`.
- When `major_change` increments, reset `minor_change` to `0`.
- If a document has no version yet, initialize it at `v1.0.0` when it first enters managed change control.
- When an existing managed document still uses the legacy two-part format, convert it to the three-part format the next time that document is updated by appending a trailing `.0` before applying the new bump logic.

## PRD Requirements

For files under `docs/01_requirements/`:

- Keep a `Document Control` section near the top with at least: `Status`, `Version`, `Last Updated`, `Owner`.
- Keep a `Change Log` section in the document.
- Every `Change Log` entry must include: `Date`, `Version`, `Change Type`, `Summary`, and `Downstream Impact`.
- Every time a PRD changes, update both the document version and the `Change Log` in the same edit.

## Cross-Role Tracking

Use `docs/03_implementation/renai-doc-sync-tracker.md` as the shared progress register for downstream work.

When requirements change:

- Update the PRD row to the new version.
- Mark impacted architecture, technical lead, and implementation rows as needing sync.
- Reset or reduce downstream progress when the upstream version has moved ahead of the tracked downstream artifact.

When architecture, implementation planning, or execution artifacts change:

- Update the relevant row in `docs/03_implementation/renai-doc-sync-tracker.md`.
- Keep these fields current: `Role`, `Artifact`, `Current Version`, `Upstream Version`, `Sync Status`, `Progress`, `Last Updated`, `Next Action`.
- Use progress percentages conservatively and only for the work represented by the artifact, not for the whole product.

## Role Expectations

- PM: maintain PRD versions and change logs, then flag downstream roles when requirements move.
- SA: track how much of the architecture set has been reconciled to the latest PRD version.
- TL: track how much of the blueprint, API contract, backlog, and task artifacts have been reconciled to the latest architecture version.
- Implementor: keep execution progress aligned with the current backlog and task artifact status values.

## Existing Execution Artifact

If `docs/03_implementation/renai-game-llm-mvp-task-artifact.json` is updated, keep its task `status` values consistent with the human-readable implementation progress recorded in `docs/03_implementation/renai-doc-sync-tracker.md`.

Do not leave a managed document update without the corresponding version, change log, or tracker maintenance when those controls apply.