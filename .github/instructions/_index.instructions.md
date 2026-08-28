---
description: Index of AI-optimized coding rules for AL development (documentation only, not auto-applied)
---

# AI Coding Rules for AL

AI-optimized AL/Business Central coding rules, maintained by the BC community.

| File | Content | Applied to |
|---|---|---|
| [al-guidelines-rules](al-guidelines-rules.instructions.md) | Core principles, quick reference, app.json/AL-Go config | `*.json` |
| [al-code-style](al-code-style.instructions.md) | Formatting, folder layout, object structure, UI strings, page fields | `*.al` |
| [al-naming-conventions](al-naming-conventions.instructions.md) | File, object, ID/affix, variable and parameter naming | `*.al` |
| [al-performance](al-performance.instructions.md) | Filtering, SetLoadFields, set-based operations | `*.al` |
| [al-error-handling](al-error-handling.instructions.md) | TryFunctions, labels, ErrorInfo, telemetry | `*.al` |
| [al-events](al-events.instructions.md) | Subscribers, integration events, thin extension objects | `*.al` |
| [al-bc-patterns](al-bc-patterns.instructions.md) | Proven BC field/object patterns | `*.al` |
| [al-testing](al-testing.instructions.md) | AL-Go structure, test generation and organization | `*.al`, `*.json` |
| [al-tdd](al-tdd.instructions.md) | Red-Green-Refactor workflow | test files |
| [al-upgrade](al-upgrade.instructions.md) | Upgrade codeunits, upgrade tags, DataTransfer | upgrade/install files |
| [README](README.instructions.md) | Contribution guide | — |

Each rule lives in exactly one file; narrow `applyTo` patterns keep upgrade and TDD rules out of the context when they are not relevant.

Repository: https://github.com/microsoft/alguidelines
