---
description: Proven BC standard patterns for common field scenarios in AL development
applyTo: "*.al"
---

# AL Business Central Patterns

## Rule 1: ID + Name field pair (lookup with readable description)

To reference another table by key but display a readable name: Integer/Code **ID field** with `TableRelation` + a separate **Text Name field** filled in the ID field's `OnValidate`. On the page show only the Name field with a custom `OnLookup`.

- **Never use a FlowField for the Name** — it does not refresh on the page without `CalcFields`, forcing fragile workarounds (page variables, `OnAfterGetRecord`, `CurrPage.Update`).
- Do not set `Editable = false` on the Name field: it would block `OnLookup`.
- In `OnLookup`, call `Rec.Validate("<...> ID", ...)` and assign `Text := Rec."<...> Name"` (BC uses `Text` to refresh the displayed value), then `exit(true)`.

```al
field(80; "Thickness Attr. ID"; Integer)
{
    TableRelation = "Item Attribute";
    trigger OnValidate()
    begin
        if ItemAttribute.Get("Thickness Attr. ID") then
            "Thickness Attr. Name" := ItemAttribute.Name
        else
            "Thickness Attr. Name" := '';
    end;
}
field(81; "Thickness Attr. Name"; Text[250]) { }
```

## Rule 2: Symbolic object references

- Always use `Page::`, `Codeunit::`, `Report::`, `Table::`, `Database::`, `Enum::` with the object name — never numeric literals, including in `[EventSubscriber]` attributes.
- Good: `Codeunit::"Upgrade Tag"`, `Database::"Sales Header"`, `Page::"My API"`. Bad: `9900`, `50061`.

## Rule 3: DataClassification on every field

- Every table field needs an explicit `DataClassification` (missing one = `ToBeClassified`, AppSourceCop AS0016).
- `SystemMetadata` (technical keys, flags) · `CustomerContent` (business data) · `EndUserIdentifiableInformation` (personal data) · `OrganizationIdentifiableInformation` · `AccountData`.
- For the classification required by telemetry see al-error-handling.instructions.md.

## Rule 4: PermissionSet objects

- Ship AL `permissionset` / `permissionsetextension` objects covering all tables of the extension (AS0103); never XML permission files.
- At least one permission set must have `Assignable = true`; use `Caption = '...', Locked = true`.
- Update the permission set whenever a table is added.
