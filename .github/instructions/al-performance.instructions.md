---
description: Performance optimization guidelines and best practices for AL development
applyTo: "*.al"
---

# AL Performance Optimization Rules

## Rule 1: Filter early

- Apply `SetRange`/`SetFilter` before iterating; never filter inside a `repeat` loop.
- Choose keys/sorting that match the filters.

## Rule 2: SetLoadFields

- Call `SetLoadFields` with only the fields actually used, **after** the filters and immediately before `Get`/`Find`.

```al
Item.SetRange("Third Party Item Exists", false);
Item.SetLoadFields("Item Category Code");
Item.FindFirst();
```

## Rule 3: In-memory structures

- Temporary tables for structured record data reused multiple times.
- `Dictionary of [...]` for key-value caches, `List of [...]` for simple collections.
- Load once, then process from memory instead of re-reading the database.

## Rule 4: Set-based operations

- Use `CalcSums`/`CalcFields` instead of manual accumulation loops.
- Use `ModifyAll`/`DeleteAll` for bulk updates; avoid nested loops.

## Rule 5: Scalability

- Assess data volume before processing; batch large datasets.
- Compute all values first, then perform a single `Modify` instead of repeated writes.
