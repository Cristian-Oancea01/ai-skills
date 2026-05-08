---
name: structured-handoff
description: Use JSON or graph-style prompt structure only for complex handoffs, orchestration, and high-constraint tasks
license: MIT
compatibility: opencode
---

## Goal

Improve clarity for complex work without adding unnecessary prompt overhead to simple tasks.

## When to use

Use this skill when the task involves:
- multi-agent or orchestrated work
- architecture or design handoff
- multi-step execution with dependencies
- many constraints, deliverables, or acceptance checks
- a need for explicit structured output

Do not use this skill for simple reads, small edits, short explanations, or straightforward fixes.

## Rule

- Plain concise prompts are the default
- Structured prompts are an exception for complex coordination

## Recommended structures

Use lightweight JSON-like fields when handing off complex work:

```json
{
  "task": "short statement",
  "constraints": ["constraint 1"],
  "targets": ["file or area"],
  "deliverable": "expected output",
  "verification": "how success will be checked"
}
```

Use graph-style stages when dependencies matter:

```text
Plan -> Explore -> Implement -> Verify -> Deliver
```

## Guardrails

- Keep the structure small; do not over-model the task
- Do not restate known context in every field
- Prefer the smallest structure that makes the handoff unambiguous
- If plain language is already clear enough, do not add structure
