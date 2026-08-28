---
description: AL Code structure, formatting, and folder organization guidelines for AL development
applyTo: "*.al"
---

# AL Code Style & Formatting Rules

## Rule 1: Formatting

- 4-space indentation (Microsoft AL standard), consistent throughout the project.
- PascalCase for objects, variables and procedures.

## Rule 2: Feature-based folder organization

- Organize by business feature: `src/<Feature>/<SubFeature>/`; shared code in `src/Common/`.
- Never group by object type (`Tables/`, `Pages/`, `Codeunits/`).

## Rule 3: Documentation

- Add XML doc comments (`<summary>`, `<param>`, `<returns>`) to global procedures of codeunits.
- Do not comment obvious operations; rely on clear naming instead.

## Rule 4: Modular code

- Small, single-purpose procedures; extract validation, calculation and posting into separate local procedures.
- Avoid monolithic procedures mixing concerns.

## Rule 5: Object internal structure (mandatory order)

1. Properties (`Access`, `Subtype`, `TableType`, ...)
2. Object constructs: table fields / page layout / actions / triggers
3. Global variables — `Label` declarations first, then other variables
4. Procedures

## Rule 6: UI string casing

- **Sentence case** when the string contains a verb: `'Post document'`, `'Calculate discount'`.
- **Title case** for noun phrases: `'Customer Name'`, `'Sales Order'`.
- Never use all-lowercase captions.

## Rule 7: Page fields and actions

- Every field and action needs `ApplicationArea` (AS0018) and `ToolTip` (AS0044) — missing ones break AppSource validation.
- ToolTip on fields starts with "Specifies …" and ends with a period; on actions it describes the effect (`'Post the sales document.'`).
- Add `Caption` whenever the property name is not already a good UI string; add `Image` and `Promoted` metadata on primary actions.
