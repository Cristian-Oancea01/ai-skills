---
name: always-plan
description: Mandatory planning rule that requires a short implementation plan before non-trivial work
license: MIT
compatibility: opencode
---

## Rule

Before implementation, always produce a short plan.

The plan must:
- state the intended change
- identify the files or areas likely to change
- state how the result will be verified
- note key constraints or risks when they materially affect the approach

## Scope

Use this for:
- coding tasks
- config changes
- scripts
- repo maintenance tasks

Skip only for:
- purely informational answers
- trivial one-step actions where the action itself is the full plan

## Plan size

Keep the plan short.

- Small task: 1-3 bullets
- Larger task: concise staged plan with phases only when they help execution

Do not turn planning into design speculation. Plan only enough to guide the implementation.

## Planning quality

- Prefer independently verifiable phases for larger changes.
- Map the plan to actual code areas, not vague subsystems.
- Name the smallest meaningful verification step for each risky change.
- Revise the plan if exploration proves an assumption wrong.

## After planning

- If the task is simple, implement directly after the plan.
- If the task is complex, use `orchestration-router` to decide whether orchestration is needed.
