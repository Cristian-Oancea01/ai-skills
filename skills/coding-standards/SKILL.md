---
name: coding-standards
description: Coding style, testing, and comment guidelines for clean, minimal, production-quality code
license: MIT
compatibility: opencode
---

## Coding style
- Keep solutions short and clear.
- Follow standard formatting and naming conventions.
- Change only what is required for the task.
- Avoid unnecessary abstractions and extra features.
- Prefer readable code over clever code.
- Read relevant files before editing them.
- Preserve existing behavior unless the task explicitly changes it.
- Reuse existing patterns and components before inventing new ones.

## Testing
- Testing is mandatory for behavior changes.
- Prefer TDD: write a failing test, implement the smallest fix, then refactor safely.
- Start with the smallest meaningful test scope, then broaden only if needed.

## Comments
- Do not add comments that restate obvious code.
- Add comments only when needed for non-trivial intent.

## Expected coding skills
- Write minimal, maintainable, production-quality code.
- Respect existing project structure and patterns.
- Apply common industry practices (readability, consistency, safety).
- Keep diffs focused: no unrelated refactors.
- Validate the exact change without expanding scope.
- Treat tests as mandatory for behavior changes.
- Follow TDD by default: failing test first, then minimal implementation.
- Prefer root-cause fixes over symptom patches.
- Remove dead code and unused imports introduced by the change.

## Examples

### Good
```js
// test first (TDD)
test('sum adds two numbers', () => {
  expect(sum(1, 2)).toBe(3);
});

function sum(a, b) {
  return a + b;
}
```

### Avoid
```js
// Adds two numbers together and returns the result.
function sum(a, b) {
  const result = a + b;
  return result;
}
```

## Continuous learning and improvement
- Regularly revisit and refine skills to reflect better practices.
- Keep all skill definitions as short as possible: remove redundancy, merge overlapping points.
- Prefer concise, high-signal rules over verbose explanations.

## Commit hygiene — sensitive data
- Never commit personal names (people, rooms, devices), non-English labels, or real credentials/entity IDs.
- Use generic English placeholders in all committed files; store real values in a private file outside the repo (e.g. `AGENTS.local.md` on the NAS, or a local secrets file).
- Before every commit: scan changed files for personal names, non-English text, passwords, IPs, and real entity IDs specific to the user's environment.

## Rule of thumb
- Smallest correct change.
- No extra code, no extra comments.
- Tests are mandatory for behavior changes.
