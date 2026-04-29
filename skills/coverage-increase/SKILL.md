---
name: coverage-increase
description: Structured workflow for raising test coverage with minimal-risk test changes that match existing project conventions
license: MIT
compatibility: opencode
---

## When to use

Use this skill when:
- a coverage gate is failing
- a report shows low-covered files or branches
- the user asks to improve automated test coverage

Do not use this skill for:
- feature work where tests are only incidental
- broad refactors disguised as test work

## Core principles

- Prefer test changes over production changes.
- Treat the coverage report as a prioritization input, not the only source of truth.
- Match existing test stack, helpers, fixtures, and naming patterns.
- Add behavior-focused tests, not implementation-detail snapshots.
- Work in small batches so failures stay local.

## Workflow

### 1. Identify targets
- gather the lowest-covered files, classes, or branches from the available report
- prioritize zero-coverage and high-risk business logic first

### 2. Learn local test conventions
- inspect existing tests near the target code
- note framework, mocking style, fixture setup, assertion style, and naming patterns

### 3. Design minimal scenarios
- list the smallest set of tests needed for happy path, key edge cases, and error paths
- prefer parameterized patterns only when the project already uses them

### 4. Implement incrementally
- update one file or small batch at a time
- run targeted tests after each batch
- stop to fix failures before adding more tests

### 5. Validate and report
- summarize changed tests, pass/fail status, and expected coverage impact
- if exact local coverage cannot be recomputed, state that clearly and point to the next verification step

## Prioritization heuristic

1. zero coverage
2. critical logic below threshold
3. medium-risk code below threshold
4. simple reliable wins when impact is similar

## Definition of done

- new or expanded tests cover meaningful behavior
- targeted tests pass locally
- changes follow existing test conventions
- no unnecessary production logic was introduced
