# ai-skills

A collection of reusable [OpenCode](https://opencode.ai) skills for AI-assisted development and home automation.

## What are skills?

Skills are `SKILL.md` files loaded into an OpenCode session to provide domain-specific context, rules, and workflows. They keep the AI focused and consistent across sessions.

## Skills

| Skill | Description |
|---|---|
| [`coding-standards`](skills/coding-standards/SKILL.md) | Coding style, testing, and comment guidelines for clean, minimal, production-quality code |
| [`agent-orchestration`](skills/agent-orchestration/SKILL.md) | Multi-agent DAG orchestration system with specialist roles for software development tasks |
| [`homeassistant-setup`](skills/homeassistant-setup/SKILL.md) | Home Assistant on QNAP NAS — Docker, integrations, Lovelace dashboard, automations |

## Usage

In OpenCode, load a skill with:

```
/skill coding-standards
```

## Conventions

- Each skill lives in `skills/<name>/SKILL.md`
- Skills are kept as short as possible — high signal, no filler
- Update skills after every session that produces new discoveries or lessons learned
- Commit skill updates in a dedicated commit: `<skill-name>: <what was learned/fixed>`
