---
name: orchestration-router
description: Mandatory routing skill that decides whether to use no orchestration, small, medium, or enterprise orchestration
license: MIT
compatibility: opencode
---

## Rule

Use this skill before loading any orchestration skill.

Its job is to classify the task into exactly one of these modes:
- `none`
- `small-task-orchestration`
- `agent-orchestration`
- `enterprise-agent-orchestration`

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

Choose `enterprise-agent-orchestration` when:
- the task is enterprise-scale or program-like
- explicit specialist role boundaries are important
- auditability and state ownership matter
- the workflow benefits from strict staged outputs and formal remediation
- you want the most deterministic orchestration mode, even with extra overhead

## Output format

Return:

```json
{
  "mode": "none | small-task-orchestration | agent-orchestration | enterprise-agent-orchestration",
  "reason": "short justification",
  "signals": ["signal 1", "signal 2"]
}
```

## Mandatory follow-through

- If `mode` is `none`, proceed without orchestration.
- If `mode` is `small-task-orchestration`, load that skill and follow it.
- If `mode` is `agent-orchestration`, load that skill and follow it.
- If `mode` is `enterprise-agent-orchestration`, load that skill and follow it.

Do not load multiple orchestration skills for the same task.
