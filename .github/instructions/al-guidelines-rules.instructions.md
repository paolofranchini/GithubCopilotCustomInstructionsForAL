---
description: AL Guidelines - Comprehensive AI-optimized coding rules for Microsoft Dynamics 365 Business Central development
applyTo: "*.al, *.json, app.json, launch.json"
---

# AL Guidelines - AI Coding Rules

Assist AL / Dynamics 365 Business Central development following these rules.

## Core principles

- Never modify standard objects: extend via extension objects and events.
- Implement application code by default; generate tests only when explicitly requested (then follow TDD).
- Keep App and Test projects separate in AL-Go workspaces (Test depends on App, never the reverse).
- Optimize for performance and handle errors explicitly.

## Summary

- **Files**: `<ObjectName>.<ObjectType>.al`; folders by feature (`src/feature/subfeature/`), not by object type.
- **Style**: 4-space indent, PascalCase; object order Properties → Constructs → Labels → Variables → Methods.
- **UI strings**: sentence case with a verb, title case for noun phrases.
- **Naming**: object names ≤ 26 chars; `Temp` prefix for temporary records; `I` prefix for interfaces, `Impl` suffix for implementations.
- **Performance**: filter early, `SetLoadFields`, `CalcSums`, temp tables/dictionaries/lists, avoid loops.
- **Errors**: `[TryFunction]`, `Label` for every user-facing text, never hardcoded strings.
- **Telemetry**: `DataClassification::SystemMetadata` (mandatory), unique prefixed EventId, PascalCase dimension keys, "Object ActionInPastTense" message.
- **Tables**: explicit `DataClassification` on every field (AS0016); ship AL `permissionset` objects covering all tables (AS0103).
- **Objects**: reference by symbol (`Page::`, `Codeunit::`, `Database::`), never numeric IDs.

Details: @al-code-style.instructions.md @al-naming-conventions.instructions.md @al-performance.instructions.md @al-error-handling.instructions.md @al-events.instructions.md @al-testing.instructions.md @al-bc-patterns.instructions.md @al-tdd.instructions.md @al-upgrade.instructions.md
