---
name: personal-assistant-lite
description: Lightweight personal assistant workflow for web-to-Obsidian tasks from OpenCode prompts
license: MIT
compatibility: opencode
---

## Goal

Execute user prompts like a practical assistant, gather information from websites, and produce clean Obsidian-ready outputs.

## Use when

- The user asks for web research, extraction, transformation, or synthesis.
- The final output should be consumed in Obsidian notes.
- The task should stay lightweight and direct.

## Workflow

1. Parse the user request into: source, target output, constraints.
2. Collect only the minimum required source content.
3. Transform into a concrete deliverable (table, checklist, summary, timeline, graph spec, or task plan).
4. Return results in Markdown that can be pasted directly into Obsidian.
5. If writing files is requested, save them in the specified vault path and keep naming explicit.

## Output rules

- Prefer concise Markdown.
- Keep headings short and useful.
- Use checklists for action items.
- Use tables for comparisons or extracted fields.
- If the user asks for a graph, output a Mermaid block unless another format is requested.

## Guardrails

- Do not fabricate facts from websites.
- Distinguish clearly between extracted facts and assumptions.
- If a website is inaccessible, report that quickly and continue with available sources.
- Ask one targeted question only when blocked by missing critical input.

## Prompt patterns

- "Visit X and extract Y into a Markdown table."
- "Compare A vs B from official docs and produce a decision checklist."
- "Read this page and create a Mermaid flowchart of the process."
- "Create an Obsidian note draft with summary, key facts, and next actions."
