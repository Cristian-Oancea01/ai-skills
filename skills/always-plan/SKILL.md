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
- Larger task: concise staged plan

Do not turn planning into design speculation. Plan only enough to guide the implementation.

## After planning

- If the task is simple, implement directly after the plan.
- If the task is complex, use `orchestration-router` to decide whether orchestration is needed.
