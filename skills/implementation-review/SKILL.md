---
name: implementation-review
description: Findings-first review workflow for completed implementations, focused on correctness, regressions, and high-signal fixes
license: MIT
compatibility: opencode
---

## When to use

Use this skill when:
- the user asks for a review of completed code
- correctness matters more than style commentary
- the implementation is broader than a trivial diff scan
- you need to validate code against requirements, plan, or expected behavior

Do not use this skill for:
- casual style feedback only
- trivial single-line edits
- architecture brainstorming before code exists

## Review posture

- Findings first. Summary second.
- Prioritize bugs, regressions, security risks, type-safety gaps, dead code, and missing tests.
- Read the relevant files, not only the diff, when surrounding context affects correctness.
- Quote concrete evidence for each finding.
- Prefer root cause over symptom description.

## Severity guide

- `critical`: runtime failure, data loss, security issue, major regression
- `high`: likely bug, broken edge case, unsafe typing at an important boundary, significant dead code
- `medium`: code smell, missing guard, weaker typing, moderate test gap
- `low`: consistency issue, follow-up cleanup, minor maintainability concern

## Workflow

### 1. Establish baseline
- identify the intended behavior from the user request, plan, tests, or nearby code
- locate the files that implement that behavior

### 2. Read implementation context
- read the full changed files when correctness depends on surrounding logic
- compare with adjacent or similar implementations to understand project conventions

### 3. Analyze by failure mode
- runtime correctness
- state management and concurrency
- nullability and typing
- security and data exposure
- dead code and unused imports
- test coverage for changed behavior

### 4. Fix or report
- if the task is review-only, report findings
- if the task includes fixing, apply the smallest correct fixes for clear high-severity issues
- verify each fix with the smallest relevant check

## Review checklist

- inputs are validated at important boundaries
- async flows cannot race or leak subscriptions/resources
- error paths are handled intentionally
- secrets or sensitive values are not introduced
- unused code introduced by the change is removed
- changed behavior has targeted test coverage
- implementation still matches stated scope

## Output contract

For each finding include:
- severity
- file and line reference
- concise issue description
- why it matters
- suggested fix or applied fix status

If no findings are discovered, say so explicitly and mention residual risks or testing gaps.
