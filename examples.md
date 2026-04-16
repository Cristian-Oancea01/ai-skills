# examples.md

## Good
```js
// test first (TDD)
test('sum adds two numbers', () => {
  expect(sum(1, 2)).toBe(3);
});

function sum(a, b) {
  return a + b;
}
```

## Avoid
```js
// Adds two numbers together and returns the result.
function sum(a, b) {
  const result = a + b;
  return result;
}
```

## Rule of thumb
- Smallest correct change.
- No extra code, no extra comments.
- Tests are mandatory for behavior changes.
