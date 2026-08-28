---
description: AL Guidelines - Comprehensive AI-optimized coding rules for Microsoft Dynamics 365 Business Central development
applyTo: "*.json"
---

# AL Guidelines - AI Coding Rules

Assist AL / Dynamics 365 Business Central development following these rules.

## Core principles

- Never modify standard objects: extend via extension objects and events.
- Implement application code by default; generate tests only when explicitly requested (then follow TDD).
- Keep App and Test projects separate in AL-Go workspaces (Test depends on App, never the reverse).
- Optimize for performance and handle errors explicitly.

## Summary

Detailed rules live in the `al-*.instructions.md` files, auto-applied to AL sources. Quick reference:

- **Files**: `<ObjectName>.<ObjectType>.al`; folders by feature (`src/feature/subfeature/`), not by object type.
- **IDs**: use `idRanges` and `mandatoryAffixes` from `app.json`; never invent object IDs.
- **Style**: 4-space indent, PascalCase; object order Properties → Constructs → Labels → Variables → Methods.
- **Naming**: object names ≤ 26 chars; `Temp` prefix for temporary records; `I` prefix for interfaces, `Impl` suffix for implementations.
- **Pages**: `ApplicationArea` and `ToolTip` on every field and action (AS0018 / AS0044).
- **Performance**: filter early, `SetLoadFields`, `CalcSums`, temp tables/dictionaries/lists, avoid loops.
- **Errors**: `[TryFunction]`, a `Label` for every user-facing text, `ErrorInfo` for actionable errors, no stray `Commit()`.
- **Tables**: explicit `DataClassification` on every field (AS0016); ship AL `permissionset` objects covering all tables (AS0103).
- **Objects**: reference by symbol (`Page::`, `Codeunit::`, `Database::`), never numeric IDs.

## app.json / AL-Go configuration

- App `app.json` must never depend on the Test project; the Test `app.json` depends on App plus the test frameworks (`Library Assert`, `Any`, `Tests-TestLibraries`).
- Keep `idRanges`, `mandatoryAffixes`, `application` and `runtime` consistent with `.AL-Go/settings.json` and `al.code-workspace`.
- Never commit credentials in `launch.json`.
