---
description: AL-Go workspace structure, test generation guidelines, and project organization rules
applyTo: "*.al,*.json"
---

# AL Testing & Project Structure Rules

## Rule 1: AL-Go workspace structure

- `App/src/...` = tables, pages, codeunits, reports, APIs, enums. `Test/src/...` = test codeunits, test pages, mocks, test data.
- Each project has its own `app.json`/`launch.json`. Never mix app logic and test code.

## Rule 2: When to generate tests

- Do not create tests unless explicitly requested ("create tests", "add test coverage", "include tests").
- When requested, follow the TDD workflow in al-tdd.instructions.md and place tests in the Test project.

## Rule 3: Dependencies

- Test `app.json` references the App project plus test frameworks (`Library Assert`, `Any`, `Tests-TestLibraries`).
- App `app.json` must never reference the Test project.

## Rule 4: Test authoring

- `Subtype = Test;` codeunits, one behaviour per `[Test]` procedure.
- Name tests `Given<Context>_When<Action>_Then<Result>` and structure the body with `// Given / // When / // Then`.
- Build data with standard libraries (`Library - Sales`, `Library - Inventory`, `Library - ERM`, `Library - Random`) instead of hardcoded values.
- Assert with `Codeunit Assert` and always pass a descriptive failure message.

## Rule 5: Test organization

- Mirror the App folder structure inside `Test/src/`; shared helpers in `Test/src/Common/TestHelpers/`.
