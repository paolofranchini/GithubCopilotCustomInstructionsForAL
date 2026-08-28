---
description: Comprehensive naming conventions for AL files, objects, variables, and functions
applyTo: "*.al"
---

# Naming Conventions Rules

## Rule 1: Object names and IDs

- PascalCase, descriptive, no cryptic abbreviations.
- Max 30 chars total, keep the name itself ≤ 26 chars to leave room for the affix.
- Good: `"Customer Ledger Entry"`, `"Sales Invoice Posting"`. Bad: `"CustLE"`, `"SIPoster"`.
- **Never invent object IDs**: take the next free ID inside `idRanges` in `app.json`, and apply the prefix/suffix declared in `mandatoryAffixes` to every new object, field, action and control.

## Rule 2: File names

- Pattern `<ObjectName>.<ObjectType>.al`, matching the object name inside the file.
- Examples: `SalesHeader.Table.al`, `CustomerCard.Page.al`, `NoSeriesImpl.Codeunit.al`, `SalesHeader.TableExt.al`, `InventorySetup.PageExt.al`, `INoSeries.Interface.al`, `NoSeriesTests.Codeunit.al`.

## Rule 3: Variables and procedures

- PascalCase, descriptive, no abbreviations unless a well-known business term.
- Record variables named after the table (`CustLedgerEntry: Record "Cust. Ledger Entry"`).
- **Temporary records must use the `Temp` prefix**: `TempSalesLine: Record "Sales Line" temporary;`.
- Procedure names start with a verb: `CalculateCustomerBalance`, `ValidateSalesDocument`.

## Rule 4: Event subscriber parameters and names

- Subscriber parameter names **must match the published event signature**: for standard table events (`OnAfterInsertEvent`, `OnBeforeModifyEvent`, ...) they are literally `Rec`, `xRec`, `RunTrigger` — renaming them does not compile.
- For integration events you publish, name the parameters after the record/table (`SalesHeader`, `Customer`, `xCustomer`), never a generic `Rec`.
- Name the subscriber procedure `<Action>On<Event><Object>`, e.g. `AddDefaultValuesOnBeforeInsertSalesHeader`.

## Rule 5: Interfaces and implementations

- Interface: `I` prefix (`ICustomerService`), file `ICustomerService.Interface.al`.
- Implementation: `Impl` suffix (`"Customer Service Impl"`), file `CustomerServiceImpl.Codeunit.al`.
