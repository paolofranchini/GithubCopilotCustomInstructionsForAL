---
description: Guidelines for implementing event-driven patterns and extensibility in AL development
applyTo: "*.al"
---

# Event-Driven Development Rules

## Rule 1: Subscribe, never modify

- Extend standard behaviour through event subscribers and extension objects only.
- Group subscribers in dedicated codeunits with the `Handler` suffix (`"Sales Document Events Handler"`); subscriber procedures are `local`.

```al
[EventSubscriber(ObjectType::Table, Database::"Sales Header", OnBeforeInsert, '', false, false)]
local procedure ValidateOnBeforeInsertSalesHeader(var SalesHeader: Record "Sales Header"; RunTrigger: Boolean)
```

## Rule 2: Publish integration events

- Raise `[IntegrationEvent(false, false)]` before/after each meaningful business step.
- Use the handled pattern: `OnBefore...(var Rec; var IsHandled)` then `if IsHandled then exit;`.
- Name events `OnBefore<Action>` / `OnAfter<Action>` and describe the intended usage in a comment.

## Rule 3: Event parameters

- Pass records `var` so subscribers can modify them.
- Include enough context (header, lines, dates, resulting document no., success flag) without passing unnecessary heavy data.
- Use descriptive parameter names matching the record/table.
