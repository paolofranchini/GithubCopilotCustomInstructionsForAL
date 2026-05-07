---
description: TDD (Red-Green-Refactor) workflow and rules for AL development
applyTo: "*.al"
---

# AL TDD (Red-Green-Refactor) Rules

These rules define a strict test-driven development workflow for AL projects in Business Central, including deploy and test execution expectations.

## Rule 1: TDD Workflow Overview (Red-Green-Refactor)

### Intent
Follow the full TDD loop for every new behavior:

1. 🔴 **Red**: write a failing test for behavior that does not exist yet
2. 🟢 **Green**: write the minimum implementation, publish the app, and run tests until they pass
3. 🔵 **Refactor**: improve code design without changing behavior, then run tests again

Never skip a phase.

## Rule 2: Write the Test First (Red Phase)

### Intent
When implementing a new feature, create the test codeunit before implementation code.

- Use descriptive Given/When/Then test names
- Assert behavior that is not implemented yet
- The test should compile but fail at runtime during the Red phase
- Keep test focus narrow: one behavior per test

## Rule 3: Minimum Implementation to Pass (Green Phase)

### Intent
After the failing test exists, implement only what is needed to make the test pass.

- Avoid over-engineering and speculative abstractions
- Add only the smallest behavior required by the failing test
- Publish to BC environment (container/sandbox via AL-Go or VS Code publish)
- Execute tests after publish (BCCT, Test Runner page, or AL-Go CI)
- Green means the relevant assertions pass consistently

## Rule 4: Deploy Workflow Before Running Tests

### Intent
Treat deployment as mandatory before test execution in BC environments.

- Publish **App** project first
- Publish **Test** project second (depends on App)
- Run tests via the standard BC test runner page (Page 130401), AL-Go `Run-Tests`, or GitHub Actions
- Record test result before moving to refactor

## Rule 5: Refactor with Safety Net (Refactor Phase)

### Intent
Refactor only after tests are green and keep behavior unchanged.

- Do not add new functionality during refactoring
- Apply safe refactors (extract procedure, rename symbols, extract constants)
- Re-run tests after each meaningful refactor step
- Stop and fix immediately if any test regresses

## Rule 6: Test Isolation and Independence

### Intent
Ensure every test is independent, deterministic, and repeatable.

- Use `[HandlerFunctions]` for dialog/confirm/message handling when needed
- Create and clean test data inside each test lifecycle (`[TearDown]`, helper codeunits, or `LibraryVariableStorage`)
- Do not depend on execution order or shared mutable state
- Keep setup explicit to avoid hidden coupling

## Rule 7: TDD in AL-Go CI/CD

### Intent
Enforce TDD quality gates in automation.

- Run automated tests in CI/CD for every pull request
- Configure AL-Go to execute tests during validation workflows
- Treat failing tests as merge blockers
- Keep pipeline and project settings aligned (for example `al.code-workspace` and `.AL-Go/settings.json`)
