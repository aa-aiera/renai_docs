---
description: "Create minimal grayscale wireframes from a PRD, HLD, blueprint, or UX flow using the Frontend Prototyper. Use when you want low-fi screen layouts, sparse UI structure, or wireframe HTML/CSS from product or architecture docs."
name: "Create Wireframe From PRD"
argument-hint: "Provide source document paths, target screens or flows, and the output path under docs/04_prototypes/."
agent: "Frontend Prototyper"
---
Create grayscale, low-fidelity wireframes from the provided source documents.

Requirements:
- Default to wireframe-first output.
- Default to minimal-information UI with only the elements needed to understand the primary screen or flow.
- Use grayscale or reduced-chrome styling unless the user explicitly asks for higher fidelity.
- Extract the primary screen, flow, and only the minimum supporting states before drafting layouts.
- Make responsive differences explicit when the source documents require both mobile and desktop behavior.
- Prefer clear layout regions, content hierarchy, navigation, and whitespace over polished visuals.
- Do not add notes files, large annotation blocks, or extra state explorations unless requested.

When helpful, produce:
- a single focused wireframe
- a small set of screen-specific wireframe files only if the flow genuinely needs more than one screen

Default output location:
- `docs/04_prototypes/`