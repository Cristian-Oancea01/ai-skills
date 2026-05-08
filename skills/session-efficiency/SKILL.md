---
name: session-efficiency
description: Session management rules for deciding when to continue context, compact it, start fresh, or avoid spawning extra agents
license: MIT
compatibility: opencode
---

## Goal

Preserve useful context and avoid paying for stale or duplicated context.

## Continue vs restart

- Continue the current session when the follow-up work is closely related and prior context is still useful
- Start a new session when the task changes topic substantially or old context is becoming noise
- Prefer continuing an existing cheap-model session over opening a new premium session for nearby follow-up work

## Compaction

- Compact when context is getting full but the thread is still relevant
- Prefer pruning tool output and stale detail before dropping high-value conclusions
- After compaction, keep a short summary of current facts, decisions, and next steps

## Context hygiene

- Carry forward conclusions, not entire logs
- Do not keep repeated copies of the same file content or long tool output in working context
- When a file was already read and has not changed, refer to the existing understanding instead of rereading it by default

## Agent spawning

- Do not spawn extra agents unless the work is meaningfully parallel or clearly benefits from specialization
- Prefer one direct edit and verification loop for small and medium tasks
- Reuse the same agent session for related follow-up work when possible
- If a spawned agent needs context, pass only the minimum needed for that branch

## Model interaction

- Avoid switching models mid-task unless there is a clear reason
- If a premium model was used for diagnosis, summarize the decision and continue routine work on a cheaper model when practical

## Rule of thumb

- Keep context warm when it helps.
- Reset when stale context costs more than it saves.
- Branch only when the branch truly earns its cost.
