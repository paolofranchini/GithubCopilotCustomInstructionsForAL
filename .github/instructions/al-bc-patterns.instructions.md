---
description: Proven BC standard patterns for common field scenarios in AL development
applyTo: "*.al"
---

# AL Business Central Patterns

Proven patterns based on how standard Business Central handles common field scenarios.

## Rule 1: ID + Name Field Pair (lookup con descrizione leggibile)

### Intent
When a setup or master table needs to reference another table by its primary key but display a human-readable name, use an Integer ID field with `TableRelation` and a separate Text Name field updated via `OnValidate`. On the page show only the Name field with a custom `OnLookup`.

**Never use FlowField for the Name** — FlowFields do not refresh on the page automatically without an explicit `CalcFields` call, requiring workarounds like page variables, `OnAfterGetRecord`, or `CurrPage.Update` that are fragile and unnecessary.

### Table pattern

```al
field(80; "Thickness Attr. ID"; Integer)
{
    Caption = 'Thickness Attribute';
    TableRelation = "Item Attribute";

    trigger OnValidate()
    var
        ItemAttribute: Record "Item Attribute";
    begin
        if ItemAttribute.Get("Thickness Attr. ID") then
            "Thickness Attr. Name" := ItemAttribute.Name
        else
            "Thickness Attr. Name" := '';
    end;
}
field(81; "Thickness Attr. Name"; Text[250])
{
    Caption = 'Thickness Attribute Name';
    // Do NOT set Editable = false here — it would prevent OnLookup from firing on the page.
    // The field is only written by the OnValidate of the ID field above.
}
```

### Page pattern

Show only the Name field bound to `Rec."... Attr. Name"`. The `OnLookup` opens the list page, calls `Rec.Validate("... Attr. ID", ...)`, and the `OnValidate` on the table updates the Name automatically. No page variables, no `OnAfterGetRecord`, no `CurrPage.Update` needed.

```al
field("Thickness Attr. Name"; Rec."Thickness Attr. Name")
{
    ApplicationArea = All;
    Caption = 'Thickness Attribute';
    ToolTip = 'Specifies the Item Attribute used for thickness.';

    trigger OnLookup(var Text: Text): Boolean
    var
        ItemAttribute: Record "Item Attribute";
        ItemAttrPage: Page "Item Attributes";
    begin
        if Rec."Thickness Attr. ID" <> 0 then
            if ItemAttribute.Get(Rec."Thickness Attr. ID") then
                ItemAttrPage.SetRecord(ItemAttribute);
        ItemAttrPage.LookupMode(true);
        if ItemAttrPage.RunModal() = Action::LookupOK then begin
            ItemAttrPage.GetRecord(ItemAttribute);
            Rec.Validate("Thickness Attr. ID", ItemAttribute.ID);
            Text := Rec."Thickness Attr. Name"; // REQUIRED: BC uses Text to update the displayed field value
            exit(true);
        end;
        exit(false);
    end;
}
```

### Anti-patterns to avoid

```al
// BAD - FlowField does not refresh automatically on the page
field(81; "Thickness Attr. Name"; Text[250])
{
    FieldClass = FlowField;
    CalcFormula = lookup("Item Attribute".Name where(ID = field("Thickness Attr. ID")));
    Editable = false;
}

// BAD - Page variable workaround needed only because of the FlowField mistake
trigger OnAfterGetRecord()
begin
    ThicknessAttrName := Rec."Thickness Attr. Name"; // fragile, unnecessary
end;
```

## Rule 2: Symbolic Object References (Page::, Codeunit::, Database::)

### Intent
Always reference AL objects by their symbolic name using the `Page::`, `Codeunit::`, `Report::`, `Database::`, `Enum::` notation instead of hardcoded numeric IDs. Symbolic references are refactor-safe, readable, and validated at compile time — numeric IDs are opaque, fragile, and break silently if objects are renumbered.

### Examples

```al
// GOOD - symbolic notation, compile-time validated
InsertTenantWebService(Page::"Fiamma B2B Sales Order API", 'fiammaB2BSalesOrders');

[EventSubscriber(ObjectType::Codeunit, Codeunit::"Upgrade Tag", 'OnGetPerCompanyUpgradeTags', '', false, false)]

if TenantWebService.Get(TenantWebService."Object Type"::Page, ServiceName) then
    exit;
```

```al
// BAD - hardcoded numeric ID, opaque and fragile
InsertTenantWebService(50061, 'FiammaB2BSalesOrders');

[EventSubscriber(ObjectType::Codeunit, 9900, 'OnGetPerCompanyUpgradeTags', '', false, false)]
```

### Anti-patterns to avoid

- Never pass a Page/Codeunit/Report ID as an integer literal where a symbolic reference is available
- Never use numeric IDs in `EventSubscriber` attributes — always use `Codeunit::"..."`, `Table::"..."` etc.
- Never use `Database::` with a numeric literal — use the table name: `Database::"Sales Header"`

## Rule 3: DataClassification on All Table Fields

### Intent
Every field in every table must have an explicit `DataClassification` property set. Fields left as `ToBeClassified` violate AppSourceCop rule AS0016 and block AppSource submission. Choose the classification that reflects the sensitivity of the data stored in that field.

**Critical for telemetry**: Only `DataClassification::SystemMetadata` is safe to include in `Session.LogMessage` calls — any other classification causes the event to be silently suppressed and never reach Application Insights.

### Examples

```al
// Good example - All fields explicitly classified
table 50100 "My Setup"
{
    fields
    {
        field(1; "Primary Key"; Code[10])
        {
            DataClassification = SystemMetadata;
        }
        field(2; "Customer Name"; Text[100])
        {
            DataClassification = CustomerContent;
        }
        field(3; "Contact Email"; Text[80])
        {
            DataClassification = EndUserIdentifiableInformation;
        }
        field(4; "API Endpoint"; Text[250])
        {
            DataClassification = SystemMetadata;
        }
    }
}
```

```al
// Bad example - Missing DataClassification (AS0016 violation)
table 50100 "My Setup"
{
    fields
    {
        field(1; "Primary Key"; Code[10]) { }       // ToBeClassified - WRONG
        field(2; "Customer Name"; Text[100]) { }    // ToBeClassified - WRONG
    }
}
```

### DataClassification reference

| Value | Use for |
|---|---|
| `SystemMetadata` | Technical keys, status flags, system-generated values — safe in telemetry |
| `CustomerContent` | User-entered business data (amounts, descriptions, dates) |
| `EndUserIdentifiableInformation` | Personal data (name, email, phone) — GDPR sensitive |
| `OrganizationIdentifiableInformation` | Company-level identifying data |
| `AccountData` | Financial account information |

## Rule 4: PermissionSet Objects for All Tables

### Intent
Every extension must ship AL `permissionset` objects that cover all tables it defines. AppSourceCop rule AS0103 enforces this. Use AL `permissionset`/`permissionsetextension` objects — not XML permission files — as they are source-controllable, version-tracked, and the current Microsoft standard. Create at least one assignable permission set per app covering all tables.

### Examples

```al
// Good example - AL PermissionSet covering all extension tables
permissionset 50100 "My App - Basic"
{
    Assignable = true;
    Caption = 'My App - Basic', Locked = true;
    Permissions =
        tabledata "My Setup" = R,
        tabledata "My Document Header" = RIMD,
        tabledata "My Document Line" = RIMD;
}
```

```al
// Good example - Extend an existing permission set
permissionsetextension 50101 "My App D365 Basic Ext" extends "D365 BASIC"
{
    Permissions =
        tabledata "My Setup" = R,
        tabledata "My Document Header" = RIMD;
}
```

```al
// Bad example - No permission set (AS0103 violation)
// App ships tables but no permissionset object — users get runtime permission errors
```

### Anti-patterns to avoid

- Never ship only XML-based permission files for new extensions — use AL `permissionset` objects
- Never set `Assignable = false` on the only permission set in an app — at least one must be assignable
- Never forget to update the permissionset when adding new tables to the data model
