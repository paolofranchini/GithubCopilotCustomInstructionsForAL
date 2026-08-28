---
description: Comprehensive naming conventions for AL files, objects, variables, and functions
applyTo: "*.al"
---

# Naming Conventions Rules

## Rule 1: Object names

- PascalCase, descriptive, no cryptic abbreviations.
- Max 30 chars total, keep the name itself ≤ 26 chars to leave room for the affix.
- Good: `"Customer Ledger Entry"`, `"Sales Invoice Posting"`. Bad: `"CustLE"`, `"SIPoster"`.

## Rule 2: File names

- Pattern `<ObjectName>.<ObjectType>.al`, matching the object name inside the file.
- Examples: `SalesHeader.Table.al`, `CustomerCard.Page.al`, `NoSeriesImpl.Codeunit.al`, `SalesHeader.TableExt.al`, `InventorySetup.PageExt.al`, `INoSeries.Interface.al`, `NoSeriesTests.Codeunit.al`.

## Rule 3: Variables and procedures

- PascalCase, descriptive, no abbreviations unless a well-known business term.
- Record variables named after the table (`CustLedgerEntry: Record "Cust. Ledger Entry"`).
- **Temporary records must use the `Temp` prefix**: `TempSalesLine: Record "Sales Line" temporary;`.
- Procedure names start with a verb: `CalculateCustomerBalance`, `ValidateSalesDocument`.

## Rule 4: Event subscriber parameters and names

- Use the table/record name as parameter name (`SalesHeader`, `Customer`, `xCustomer`), never generic `Rec`.
- Name the subscriber `<Action>On<Event><Object>`, e.g. `AddDefaultValuesOnBeforeInsertSalesHeader`.

## Rule 5: Interfaces and implementations

- Interface: `I` prefix (`ICustomerService`), file `ICustomerService.Interface.al`.
- Implementation: `Impl` suffix (`"Customer Service Impl"`), file `CustomerServiceImpl.Codeunit.al`.
