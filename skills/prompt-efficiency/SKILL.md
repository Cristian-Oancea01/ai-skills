---
name: prompt-efficiency
description: Prompt and context shaping rules that improve result quality while reducing token and cache usage
license: MIT
compatibility: opencode
---

## Goal

Reduce token usage without reducing correctness.

## Prompt shaping

- Rewrite the working prompt in your head before acting: keep only the task, constraints, target files or surfaces, and verification step
- Remove repeated background once the task is clear
- Prefer one precise request over multiple overlapping asks
- Use concrete file paths, function names, commands, and error strings when known
- Ask at most one short clarifying question when a missing decision blocks correct execution

## Context control

- Read only the files needed for the current step
- Avoid rereading large files unless they changed or more context is required
- Prefer targeted search over broad exploration
- Summarize intermediate conclusions instead of replaying long reasoning chains
- Do not resend large logs or full files when a small excerpt is enough

## Session efficiency

- Keep related follow-up work in the same session when context reuse helps
- Start a new session when the topic changes substantially or stale context is becoming noise
- Avoid unnecessary model switches because they reduce cache reuse
- Prefer one direct edit and verification loop over repeated speculative cycles

## Tool and subagent prompts

- Pass only the minimum context needed for the subtask
- Name the exact deliverable you need back
- Avoid broad instructions when a narrow objective will do

## Output style

- Be concise by default
- Expand only when correctness, reviewability, or the user request requires it
