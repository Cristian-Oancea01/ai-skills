# ai-skills

A collection of reusable [OpenCode](https://opencode.ai) skills for AI-assisted development.

## What are skills?

Skills are `SKILL.md` files loaded into an OpenCode session to provide domain-specific context, rules, and workflows. They keep the AI focused and consistent across sessions.

## Skills

| Skill | Description |
|---|---|
| [`coding-standards`](skills/coding-standards/SKILL.md) | Coding style, testing, and comment guidelines for clean, minimal, production-quality code |
| [`agent-orchestration`](skills/agent-orchestration/SKILL.md) | Multi-agent DAG orchestration system with specialist roles for software development tasks |

## Agent Orchestration — DAG Overview

```mermaid
flowchart TD
    U([User Task]) --> O[Orchestrator_Prime\nbuild DAG]

    O --> RA[Requirement_Analyst]
    RA --> CE[Codebase_Explorer]
    CE --> SA[Solution_Architect]
    SA --> CI[Code_Implementer]

    CI --> TE[Test_Engineer]
    CI --> CR[Code_Reviewer]

    TE --> CA[Critic_Agent]
    CR --> CA

    CA -->|pass| D([Deliver])
    CA -->|fail, cycle ≤ 3| O
    CA -->|fail, cycle > 3| ESC([Escalate to User])
```

Each node is an independent specialist agent. All data flows through a shared Global State object — agents never call each other directly.

## Usage

In OpenCode, load a skill with:

```
/skill coding-standards
```

Or reference it from your `AGENTS.md`:

```md
- `coding-standards` — always load for any coding task
- `agent-orchestration` — load for complex multi-step tasks
```

## Conventions

- Each skill lives in `skills/<name>/SKILL.md`
- Skills are kept as short as possible — high signal, no filler
- Update skills after every session that produces new discoveries or lessons learned
- Commit skill updates in a dedicated commit: `<skill-name>: <what was learned/fixed>`
