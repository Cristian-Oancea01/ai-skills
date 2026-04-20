---
name: project-bootstrap
description: On first contact with any project, check for AGENTS.md and create it if missing. Ensures every project has a single-source reference file so future sessions never need to re-explore the codebase.
license: MIT
compatibility: opencode
---

## Rule

At the start of every session involving a project:

1. Check if `AGENTS.md` exists at the project root.
2. **If it exists** — read it. Do not explore the codebase. Use it as the sole source of truth for structure, entities, conventions, and file locations.
3. **If it does not exist** — explore the project thoroughly, then create `AGENTS.md` before doing any other work. Commit and push it immediately with message `docs: add AGENTS.md project reference`.

## What AGENTS.md must contain

- Directory structure (annotated tree)
- All views / screens / pages with file paths and purpose
- All key entities, IDs, device names, and their locations/rooms
- All integrations and the method used
- All automations with scope (room vs house-wide)
- Conventions: naming, placeholder IDs, secrets policy, known limitations

## Format rules

- Use tables for entities and file listings — scannable at a glance.
- Mark placeholder/template values clearly (e.g. `# replace`).
- Document known limitations inline (e.g. "sound beep not supported by integration X").
- Keep it current: update AGENTS.md in the same commit as any structural change.

## Why

Re-exploring a codebase on every session wastes tokens and time. A well-maintained `AGENTS.md` reduces session startup from many tool calls to a single file read.
