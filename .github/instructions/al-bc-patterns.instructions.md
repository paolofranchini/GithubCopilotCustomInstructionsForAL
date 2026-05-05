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
