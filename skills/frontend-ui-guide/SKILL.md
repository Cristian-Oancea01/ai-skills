---
name: frontend-ui-guide
description: Frontend implementation guide that prioritizes reuse of existing components, patterns, and design tokens before creating new UI
license: MIT
compatibility: opencode
---

## When to use

Use this skill for:
- building or modifying frontend UI
- component, template, and layout work
- reviewing whether a UI change fits the existing design system

Do not use this skill for:
- backend-only work
- visual redesigns that intentionally replace the current system

## Core rules

- Use existing components first.
- Read a component's implementation or documented API before using it.
- Check for nearby examples before inventing markup.
- Create a new component only when no suitable pattern exists and reuse is likely.
- Preserve the established visual language unless the task explicitly changes it.

## Workflow

### 1. Identify the UI need
- classify the need: form control, table, dialog, navigation, card, status display, layout, or interaction pattern

### 2. Search for reusable pieces
- look for shared components, existing templates, and similar screens
- prefer the closest existing pattern over raw HTML if a reusable component already exists

### 3. Read before reuse
- inspect inputs, outputs, props, slots, events, and styling hooks
- verify how state, validation, and accessibility are handled

### 4. Implement conservatively
- compose from existing pieces first
- follow current spacing, typography, color, and responsive conventions
- avoid one-off styling when a design token or utility already exists

### 5. Verify
- check desktop and mobile behavior
- verify basic accessibility states such as labels, focus, disabled state, and error messaging

## Review checklist

- no new bespoke component where an existing one would work
- no guessed component API
- no hardcoded design values when tokens or utilities exist
- responsive behavior remains intact
- accessibility basics are preserved

## Rule of thumb

- Reuse, then adapt, then create.
