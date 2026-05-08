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
| `cost-optimized-defaults` | Convenience bundle for model, prompt, and session efficiency |
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
| `structured-handoff` | Use structured JSON or graph prompts only for complex coordination |

## Recommended default stack

For most coding sessions, load:
- `project-bootstrap`
- `always-plan`
- `orchestration-router`
- `coding-standards`

Add these when efficiency matters:
- `cost-optimized-defaults`
- `model-selection`
- `prompt-efficiency`
- `session-efficiency`

Add this when the task has complex handoffs or many constraints:
- `structured-handoff`

If you prefer one skill instead of three, load `cost-optimized-defaults`.

## Efficiency workflow

Use this sequence when you want better cost and cache behavior:

1. Load `model-selection` to route the task to the cheapest capable model.
2. Load `prompt-efficiency` to shorten prompts, reduce repeated context, and keep tool requests narrow.
3. Load `session-efficiency` to decide whether to continue, compact, restart, or avoid extra agents.
4. Use `orchestration-router` before any orchestration skill.
5. Prefer `small-task-orchestration` for narrow tasks instead of heavier multi-agent flows.

Or use `cost-optimized-defaults` to apply steps 1 to 3 together.

Use `structured-handoff` only when explicit JSON or graph structure will make a complex handoff clearer.

## Free-model opt-out

`model-selection` is generic and does not assume any specific provider.

If the user says to avoid free models, the skill instructs the agent to avoid recommending them for the session and prefer lower-cost paid models when model choice is available.

OpenCode supports manual model selection through config, startup flags, and `/models`. These skills should treat model choice as user- or runtime-controlled unless the current environment explicitly supports switching.

Third-party plugins may add automatic routing, fallback, or context-based model switching. Recommendations in this repo should still make sense without any plugin, and plugin-specific automation should be treated as optional support for the same policy rather than the default behavior.

## Optional Plugins

These skills work without plugins. In a default OpenCode setup:
- pick models manually through config, startup flags, or `/models`
- recommend a model change explicitly when the current model is no longer a good fit

If you want automatic support for the same policy, these plugins are a practical pair:
- `opencode-auto-fallback` — automatic model switching for provider errors, rate limits, and some context-related failures
- `opencode-model-router` — automatic tier-based routing across cheap, normal, and heavy models

### Install plugins

Use the OpenCode plugin installer:

```bash
# Global install
opencode plugin opencode-auto-fallback -g
opencode plugin opencode-model-router -g
```

Or add them manually to `~/.config/opencode/opencode.json`:

```json
{
  "plugin": [
    "opencode-auto-fallback",
    "opencode-model-router"
  ]
}
```

### Configure `opencode-auto-fallback`

Create `~/.config/opencode/fallback.json` and enable it:

```json
{
  "enabled": true,
  "defaultFallback": [
    "cheap-backup-model",
    "stronger-backup-model"
  ],
  "cooldownMs": 60000,
  "maxRetries": 2
}
```

Usage notes:
- use it for resilience first: rate limits, provider failures, transient API issues
- keep the fallback chain aligned with the same cheapest-capable policy from these skills
- prefer a cheap backup before a premium one unless the task truly needs the premium model

### Configure `opencode-model-router`

Create `~/.config/opencode/tiers.json` with a preset that matches your available providers and subscriptions:

```json
{
  "activePreset": "my-preset",
  "activeMode": "budget",
  "presets": {
    "my-preset": {
      "fast": { "model": "cheap-model", "costRatio": 1 },
      "medium": { "model": "default-coding-model", "costRatio": 5 },
      "heavy": { "model": "strong-debug-model", "costRatio": 20 }
    }
  }
}
```

Usage notes:
- `fast` should be your cheapest reliable read/search model
- `medium` should be your normal implementation model
- `heavy` should be reserved for debugging, architecture, and risky multi-file analysis
- use a budget-oriented mode by default if credit savings matter more than maximum quality

### Recommended behavior with or without plugins

- Without plugins: recommend model changes explicitly and continue with the current model when the work is still safe
- With `opencode-auto-fallback`: let failures and rate limits switch models automatically, but keep the fallback order cost-aware
- With `opencode-model-router`: let routing automate model choice, but keep the tier mapping aligned with the same skill policy
- With both plugins: use the router for normal task selection and the fallback plugin for failure recovery

## Repository structure

```text
ai-skills/
├── AGENTS.md
├── README.md
└── skills/
    ├── agent-orchestration/
    ├── always-plan/
    ├── coding-standards/
    ├── cost-optimized-defaults/
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
    ├── small-task-orchestration/
    └── structured-handoff/
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
- `cost-optimized-defaults` — load model, prompt, and session efficiency together
- `model-selection` — route to cheaper capable models first
- `prompt-efficiency` — reduce token and cache usage through tighter prompts
- `session-efficiency` — keep useful context and drop stale context cheaply
- `structured-handoff` — use structured prompts only when plain language is not enough
```

## Maintenance

- Keep skills short and high signal
- Edit an existing skill when the new rule is inseparable from its purpose
- Add a new skill when the workflow has a distinct trigger or objective
- Update `AGENTS.md` and this `README.md` whenever the skill inventory changes
- Use dedicated commits for skill changes
