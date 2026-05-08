# ai-skills

Reusable OpenCode skills for planning, implementation, orchestration, review, and efficiency.

## What this repo is for

Each skill lives in `skills/<name>/SKILL.md` and is meant to be loaded into an OpenCode session as focused behavior.

The repo emphasizes:
- small correct changes
- clear orchestration boundaries
- reviewable workflows
- lower token and cache usage
- cost-aware model selection

## Skill catalog

| Skill | Purpose |
| --- | --- |
| `agent-orchestration` | Medium orchestration for larger multi-step software work |
| `always-plan` | Require a short plan before non-trivial implementation |
| `coding-standards` | Minimal, production-quality coding and testing rules |
| `coverage-increase` | Improve automated coverage in low-risk batches |
| `enterprise-agent-orchestration` | Heavyweight orchestration for enterprise-scale work |
| `frontend-ui-guide` | Reuse-first frontend implementation guidance |
| `implementation-review` | Findings-first review workflow |
| `model-selection` | Cost-aware model routing with optional free-model opt-out |
| `orchestration-router` | Decide whether to use no orchestration, small, medium, or enterprise orchestration |
| `project-bootstrap` | Read `AGENTS.md` first or create it if missing |
| `prompt-efficiency` | Tighten prompts and context to reduce token and cache usage |
| `research-agent` | Evidence-first workflow for market and competitor research |
| `session-efficiency` | Manage continuation, compaction, fresh starts, and agent fanout efficiently |
| `small-task-orchestration` | Lightweight plan-explore-implement-verify flow |

## Recommended default stack

For most coding sessions, load:
- `project-bootstrap`
- `always-plan`
- `orchestration-router`
- `coding-standards`

Add these when efficiency matters:
- `model-selection`
- `prompt-efficiency`
- `session-efficiency`

## Efficiency workflow

Use this sequence when you want better cost and cache behavior:

1. Load `model-selection` to route the task to the cheapest capable model.
2. Load `prompt-efficiency` to shorten prompts, reduce repeated context, and keep tool requests narrow.
3. Load `session-efficiency` to decide whether to continue, compact, restart, or avoid extra agents.
4. Use `orchestration-router` before any orchestration skill.
5. Prefer `small-task-orchestration` for narrow tasks instead of heavier multi-agent flows.

## Free-model opt-out

`model-selection` is generic and does not assume any specific provider.

If the user says to avoid free models, the skill instructs the agent to disable free-model routing for the session and fall back to paid cheap models first.

## Repository structure

```text
ai-skills/
├── AGENTS.md
├── README.md
└── skills/
    ├── agent-orchestration/
    ├── always-plan/
    ├── coding-standards/
    ├── coverage-increase/
    ├── enterprise-agent-orchestration/
    ├── frontend-ui-guide/
    ├── implementation-review/
    ├── model-selection/
    ├── orchestration-router/
    ├── project-bootstrap/
    ├── prompt-efficiency/
    ├── research-agent/
    ├── session-efficiency/
    └── small-task-orchestration/
```

## Install

Clone into the OpenCode global skills directory.

```bash
# macOS / Linux
git clone https://github.com/Cristian-Oancea01/ai-skills.git ~/.config/opencode/skills

# Windows
git clone https://github.com/Cristian-Oancea01/ai-skills.git %APPDATA%\opencode\skills
```

OpenCode discovers skills from `skills/*/SKILL.md`.

## Usage

Load a skill manually:

```text
/skill coding-standards
```

Reference them from `AGENTS.md`:

```md
- `project-bootstrap` — read `AGENTS.md` first
- `always-plan` — make a short plan before non-trivial work
- `orchestration-router` — choose no orchestration, small, medium, or enterprise
- `coding-standards` — keep code changes minimal and verified
- `model-selection` — route to cheaper capable models first
- `prompt-efficiency` — reduce token and cache usage through tighter prompts
- `session-efficiency` — keep useful context and drop stale context cheaply
```

## Maintenance

- Keep skills short and high signal
- Edit an existing skill when the new rule is inseparable from its purpose
- Add a new skill when the workflow has a distinct trigger or objective
- Update `AGENTS.md` and this `README.md` whenever the skill inventory changes
- Use dedicated commits for skill changes
