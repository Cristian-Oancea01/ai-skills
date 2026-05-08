---
name: cost-optimized-defaults
description: Convenience skill that applies cost-aware model selection, prompt efficiency, and session efficiency together for normal coding sessions
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
- `model-selection`
- `prompt-efficiency`
- `session-efficiency`

If those skills are available, load them in that order.

## Combined behavior

- Route work to the cheapest capable model first
- Allow free models for lightweight work unless the user opts out
- Keep prompts short, concrete, and non-repetitive
- Reuse useful session context and avoid stale context
- Avoid unnecessary agent spawning and model switching
- Escalate to premium models only when the task clearly needs it or a cheaper attempt failed

## User preference

- If the user says to avoid free models, disable free-model routing for the session
- If the user asks for a specific provider or model, follow that request
- If reliability, privacy, or latency requirements conflict with cheaper routing, prefer the user requirement

## Rule of thumb

- Cheap by default
- Precise by default
- Reuse context when it helps
- Escalate only when earned
