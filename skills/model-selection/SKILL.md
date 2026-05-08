---
name: model-selection
description: Cost-aware model routing that defaults to cheaper capable models and allows opting out of free-model usage
license: MIT
compatibility: opencode
---

## Goal

Choose the cheapest model that can plausibly succeed.

## Rule

Before using or switching models:
- classify the task by risk, ambiguity, and context size
- start with a cheaper capable model
- escalate only after one reasonable failure or when the task is clearly high risk

## Routing

- Search, reading, simple explanations: cheapest lightweight model available
- Small edits, refactors, tests, config changes: cheap coding model
- Broad summarization and large-context comparison: cheap summarization model first, stronger long-context model only if needed
- Tricky debugging, risky multi-file changes, architecture: premium model

## Free models

- Free models are allowed for lightweight tasks when they are good enough for the job
- Do not use free models for risky code changes unless they are already proven reliable in that workflow
- If the user says to avoid free models, treat them as disabled for the session
- Respect explicit user preferences about privacy, providers, latency, or reliability over cost

## Escalation

- One cheap attempt first
- Escalate one tier at a time
- After diagnosis on a premium model, drop back to a cheaper model for routine implementation when practical

## Guardrails

- Do not switch models mid-task without a clear reason
- Do not use premium models for routine search, reading, formatting, or straightforward CRUD work
- Prefer stable defaults over constantly re-optimizing for tiny quality gains
