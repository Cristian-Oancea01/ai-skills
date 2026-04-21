# AGENTS.md — Project Reference for AI Agents

Read this file first before exploring any other file in this repo.
It documents the repository structure, skill inventory, and maintenance rules.
Update this file whenever the repo structure or skill set changes.

---

## Project Overview

`ai-skills` is a repository of reusable global OpenCode skills.
Each skill lives in `skills/<name>/SKILL.md` and is designed to be loaded into sessions as focused guidance.

---

## Directory Structure

```
ai-skills/
├── .github/
│   └── workflows/
│       ├── README.md              # Notes for GitHub workflow usage
│       └── opencode.yml           # GitHub Action entrypoint for OpenCode
├── skills/
│   ├── agent-orchestration/
│   │   └── SKILL.md               # Full DAG orchestration for larger multi-step work
│   ├── always-plan/
│   │   └── SKILL.md               # Planning-first rule before non-trivial implementation
│   ├── coding-standards/
│   │   └── SKILL.md               # Minimal coding, testing, and commit hygiene rules
│   ├── orchestration-router/
│   │   └── SKILL.md               # Decides whether to use orchestration and which mode
│   └── project-bootstrap/
│       └── SKILL.md               # AGENTS.md-first workflow for projects
│   └── small-task-orchestration/
│       └── SKILL.md               # Lightweight orchestration for small tasks and scripts
├── AGENTS.md                      # This file
├── LICENSE
└── README.md                      # Repo overview and usage
```

---

## Skill Inventory

| Skill | Path | Purpose |
|---|---|---|
| `always-plan` | `skills/always-plan/SKILL.md` | Require a short plan before non-trivial implementation work |
| `coding-standards` | `skills/coding-standards/SKILL.md` | Smallest-correct-change coding rules, testing, and sensitive-data hygiene |
| `orchestration-router` | `skills/orchestration-router/SKILL.md` | Decide whether to use no orchestration, small-task orchestration, or full orchestration |
| `project-bootstrap` | `skills/project-bootstrap/SKILL.md` | Require `AGENTS.md` lookup first for project work |
| `agent-orchestration` | `skills/agent-orchestration/SKILL.md` | Full multi-agent DAG for larger software tasks |
| `small-task-orchestration` | `skills/small-task-orchestration/SKILL.md` | Lightweight plan-explore-implement-verify flow for small tasks and scripts |

---

## Conventions

- Keep skills short, direct, and high signal.
- Prefer adding a new skill when it has a distinct trigger or workflow.
- Prefer editing an existing skill when the new rule is inseparable from its current purpose.
- Keep examples minimal and operational.
- Commit skill changes with dedicated, descriptive commits.

---

## Known Constraints

- The repo contains skills only; there is no runtime application code.
- `README.md` should stay aligned with the actual skill inventory.
- Any new skill added under `skills/` should also be reflected here.
