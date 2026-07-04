---
name: cost-optimized-defaults
description: Convenience skill that applies prompt efficiency and session efficiency together for normal coding sessions
license: MIT
compatibility: opencode
---

## When to use

Use this skill for normal coding sessions when you want better cost efficiency without manually loading multiple efficiency skills.

Do not use this skill when:
- the user explicitly prefers a specific model strategy
- cost is not a concern for the session
- the task requires a provider or policy that conflicts with these defaults

## Rule

When this skill is loaded, apply the behavior of:
- `prompt-efficiency`
- `session-efficiency`

If those skills are available, load them in that order.

## Combined behavior

- Keep prompts short, concrete, and non-repetitive
- Use structured JSON or graph handoff only for complex coordination or high-constraint tasks
- Reuse useful session context and avoid stale context
- Avoid unnecessary agent spawning

## User preference

- If the user asks for a specific provider or model, follow that request

## Environment reality

- Do not assume optional plugins are installed.
- Keep behavior stable in default OpenCode setups.

## Rule of thumb

- Precise by default
- Reuse context when it helps
