---
description: TDD (Red-Green-Refactor) workflow and rules for AL development
applyTo: "*.al"
---

# AL TDD (Red-Green-Refactor) Rules

Apply this workflow only when the user explicitly asks for tests. Never skip a phase.

## 1. Red — test first

- Write the failing test before any implementation code.
- Descriptive Given/When/Then name, one behaviour per test; it must compile and fail at runtime.

## 2. Green — minimum implementation

- Implement only what the failing test requires; no speculative abstractions.
- Publish App first, then Test, then run the tests until they pass consistently.

## 3. Refactor

- Only with green tests; behaviour must stay unchanged (extract procedure, rename, extract constants).
- Re-run tests after each refactor step; fix regressions immediately.

## 4. Deploy & run

- Publishing is mandatory before test execution (App → Test).
- Run via test runner page 130401, AL-Go `Run-Tests`, or GitHub Actions.

## 5. Isolation

- Each test independent, deterministic, order-agnostic.
- Use `[HandlerFunctions]` for dialogs/confirms/messages, create and clean data within the test lifecycle, keep setup explicit.

## 6. CI/CD

- Run tests on every pull request; failing tests block the merge.
- Keep `al.code-workspace` and `.AL-Go/settings.json` aligned.
