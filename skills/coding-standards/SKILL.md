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

## Testing
- Testing is mandatory for behavior changes.
- Prefer TDD: write a failing test, implement the smallest fix, then refactor safely.

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

## Rule of thumb
- Smallest correct change.
- No extra code, no extra comments.
- Tests are mandatory for behavior changes.
