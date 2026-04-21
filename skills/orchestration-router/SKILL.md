---
name: orchestration-router
description: Mandatory routing skill that decides whether to use no orchestration, small-task orchestration, or full agent orchestration
license: MIT
compatibility: opencode
---

## Rule

Use this skill before loading any orchestration skill.

Its job is to classify the task into exactly one of these modes:
- `none`
- `small-task-orchestration`
- `agent-orchestration`

## Decision rubric

Choose `none` when:
- the task is informational only
- the task is trivial and can be completed as a single obvious action
- orchestration would add more overhead than value

Choose `small-task-orchestration` when:
- the task is implementation work but narrow in scope
- likely touches one or two files
- the path is clear after brief context reading
- one verification step is enough

Choose `agent-orchestration` when:
- the task has multiple dependent stages
- design decisions need to be separated from implementation
- tests and review should run as distinct phases
- the task may branch into parallel or partially independent work
- failures likely require a structured remediation loop

## Output format

Return:

```json
{
  "mode": "none | small-task-orchestration | agent-orchestration",
  "reason": "short justification",
  "signals": ["signal 1", "signal 2"]
}
```

## Mandatory follow-through

- If `mode` is `none`, proceed without orchestration.
- If `mode` is `small-task-orchestration`, load that skill and follow it.
- If `mode` is `agent-orchestration`, load that skill and follow it.

Do not load both orchestration skills for the same task.
