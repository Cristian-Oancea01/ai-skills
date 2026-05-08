---
name: model-selection
description: Cost-aware model routing that defaults to cheaper capable models and allows opting out of free-model usage
license: MIT
compatibility: opencode
---

## Goal

Choose or recommend the cheapest model that can plausibly succeed.

## Rule

Before selecting or recommending a model:
- classify the task by risk, ambiguity, and context size
- prefer a cheaper capable model first
- escalate only after one reasonable failure or when the task is clearly high risk
- do not assume the current session can switch models programmatically

## Default vs plugins

- In default OpenCode setups, model selection is manual or config-driven
- Some third-party plugins add automatic routing, fallback, or context-based model changes
- Write recommendations so they still work without a plugin: recommend the right model first, then use plugin automation only when it is actually installed

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
- If the environment supports model changes, prefer using stronger models only for the work that needs them
- If the environment does not support model changes, tell the user which model to switch to and why
- If a routing or fallback plugin is installed, configure it to follow the same cheapest-capable escalation order

## Guardrails

- Do not assume automatic runtime model switching
- Do not change models without a clear reason when the environment supports switching
- Do not use premium models for routine search, reading, formatting, or straightforward CRUD work
- Prefer stable defaults over constantly re-optimizing for tiny quality gains
