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
│   │   └── SKILL.md               # Medium orchestration for larger multi-step work
│   ├── always-plan/
│   │   └── SKILL.md               # Planning-first rule before non-trivial implementation
│   ├── coding-standards/
│   │   └── SKILL.md               # Minimal coding, testing, and commit hygiene rules
│   ├── cost-optimized-defaults/
│   │   └── SKILL.md               # Convenience bundle for model, prompt, and session efficiency
│   ├── coverage-increase/
│   │   └── SKILL.md               # Minimal-risk workflow for improving automated test coverage
│   ├── enterprise-agent-orchestration/
│   │   └── SKILL.md               # Heavyweight orchestration for enterprise-scale work
│   ├── frontend-ui-guide/
│   │   └── SKILL.md               # Reuse-first frontend UI composition and review guidance
│   ├── implementation-review/
│   │   └── SKILL.md               # Findings-first review workflow for completed implementations
│   ├── personal-assistant-lite/
│   │   └── SKILL.md               # Lightweight web-to-Obsidian assistant workflow
│   ├── orchestration-router/
│   │   └── SKILL.md               # Decides whether to use orchestration and which mode
│   ├── prompt-efficiency/
│   │   └── SKILL.md               # Prompt and context shaping for lower token and cache usage
│   ├── project-bootstrap/
│   │   └── SKILL.md               # AGENTS.md-first workflow for projects
│   ├── research-agent/
│   │   ├── SKILL.md               # Evidence-first workflow for market and opportunity research
│   │   ├── examples/
│   │   │   └── example-opportunity-report.md
│   │   └── templates/
│   │       ├── opportunity-report.md
│   │       ├── research-brief.md
│   │       └── source-log.md
│   ├── session-efficiency/
│   │   └── SKILL.md               # Session continuation, compaction, and agent-spawning efficiency rules
│   ├── structured-handoff/
│   │   └── SKILL.md               # Selective JSON or graph-style handoff for complex coordination tasks
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
| `cost-optimized-defaults` | `skills/cost-optimized-defaults/SKILL.md` | Apply model selection, prompt efficiency, and session efficiency together for normal coding sessions |
| `coverage-increase` | `skills/coverage-increase/SKILL.md` | Improve automated test coverage in small, convention-matching batches |
| `enterprise-agent-orchestration` | `skills/enterprise-agent-orchestration/SKILL.md` | Heavyweight orchestration for enterprise-scale work with explicit role and state boundaries |
| `frontend-ui-guide` | `skills/frontend-ui-guide/SKILL.md` | Prefer existing frontend components, tokens, and patterns before creating new UI |
| `implementation-review` | `skills/implementation-review/SKILL.md` | Findings-first review of completed implementations, focused on correctness and regressions |
| `personal-assistant-lite` | `skills/personal-assistant-lite/SKILL.md` | Lightweight workflow for web research/extraction and Obsidian-ready outputs |
| `orchestration-router` | `skills/orchestration-router/SKILL.md` | Decide whether to use no orchestration, small, medium, or enterprise orchestration |
| `prompt-efficiency` | `skills/prompt-efficiency/SKILL.md` | Reduce token and cache usage by tightening prompts, context, and tool instructions |
| `project-bootstrap` | `skills/project-bootstrap/SKILL.md` | Require `AGENTS.md` lookup first for project work |
| `research-agent` | `skills/research-agent/SKILL.md` | Evidence-first workflow for market scans, competitor review, and opportunity research |
| `session-efficiency` | `skills/session-efficiency/SKILL.md` | Decide when to continue, compact, restart, or avoid extra agents to preserve useful context cheaply |
| `structured-handoff` | `skills/structured-handoff/SKILL.md` | Use JSON or graph-style prompts only where structured coordination is worth the extra input |
| `agent-orchestration` | `skills/agent-orchestration/SKILL.md` | Medium orchestration for larger software tasks |
| `small-task-orchestration` | `skills/small-task-orchestration/SKILL.md` | Lightweight plan-explore-implement-verify flow for small tasks and scripts |

---

## Conventions

- Keep skills short, direct, and high signal.
- Prefer adding a new skill when it has a distinct trigger or workflow.
- Prefer editing an existing skill when the new rule is inseparable from its current purpose.
- Keep examples minimal and operational.
- Commit skill changes with dedicated, descriptive commits.

---

## Optional Plugins

- This repo should not depend on model-routing plugins.
- Skills must remain usable in default OpenCode setups without plugin-based routing assumptions.

---

## Known Constraints

- The repo contains skills only; there is no runtime application code.
- `README.md` should stay aligned with the actual skill inventory.
- Any new skill added under `skills/` should also be reflected here.
