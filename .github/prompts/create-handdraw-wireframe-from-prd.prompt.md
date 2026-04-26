---
description: "Create minimal hand-drawn sketch wireframes from a PRD, HLD, blueprint, or UX flow using the Frontend Prototyper. Use when you want rough sparse screen layouts, notebook-style wireframes, or sketch HTML/CSS from product or architecture docs."
name: "Create Handdraw Wireframe From PRD"
argument-hint: "Provide source document paths, target screens or flows, and the output path under docs/04_prototypes/."
agent: "Frontend Prototyper"
---
Create hand-drawn, sketch-style wireframes from the provided source documents.

Requirements:
- Default to low-fidelity hand-drawn wireframe output.
- Default to minimal-information UI with only the core screen structure and primary actions visible.
- Use grayscale, rough borders, annotation-friendly layout regions, and reduced visual polish.
- Extract the primary screen, flow, and only the minimum supporting states before drafting layouts.
- Make responsive differences explicit when the source documents require both mobile and desktop behavior.
- Prefer screen structure, navigation, whitespace, and a few necessary labels over realistic UI polish.
- Keep annotations light and only add them when they materially clarify the sketch.

When helpful, produce:
- a single focused wireframe
- a small set of hand-drawn wireframe files only if the flow genuinely needs more than one screen

Default output location:
- `docs/04_prototypes/`