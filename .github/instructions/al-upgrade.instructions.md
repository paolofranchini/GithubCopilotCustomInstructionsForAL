---
description: Guidelines for writing and handling upgrade code
applyTo: "*.al"
---

# AL Upgrade Code Rules

## 1. Upgrade codeunit structure

- `Subtype = Upgrade;` with `OnUpgradePerCompany` / `OnUpgradePerDatabase` triggers containing **only calls** to local procedures — never direct implementation.
- Do **not** use `OnValidateUpgradePer*` or `OnCheckPreconditionsPer*` triggers (they run on every upgrade). If unavoidable: written justification + upgrade-tag guard to skip when already upgraded.

## 2. Error handling

- Never block the upgrade: throw errors only when strictly necessary.
- Protect **every** read with `if ... then`: `if Item.Get() then`, `if Customer.FindSet() then`, `if not Vendor.FindLast() then exit;`.
- Log unexpected situations with telemetry (`Session.LogMessage`, `DataClassification::SystemMetadata`) and continue.

## 3. Execution control — upgrade tags, not version checks

- Never branch on `DataVersion()` inside upgrade code. The only accepted version check is detecting a fresh install in `OnInstallAppPerCompany` (`DataVersion = '0.0.0.0'` → set all upgrade tags).

```al
local procedure UpgradeMyFeature()
begin
    if UpgradeTag.HasUpgradeTag(MyUpgradeTag()) then
        exit;
    // upgrade code
    UpgradeTag.SetUpgradeTag(MyUpgradeTag());
end;

[EventSubscriber(ObjectType::Codeunit, Codeunit::"Upgrade Tag", 'OnGetPerCompanyUpgradeTags', '', false, false)]
local procedure RegisterPerCompanyTags(var PerCompanyUpgradeTags: List of [Code[250]])
begin
    PerCompanyUpgradeTags.Add(MyUpgradeTag());
end;
```

- Every new tag **must** be registered, in `OnGetPerCompanyUpgradeTags` if called from `OnUpgradePerCompany`, in `OnGetPerDatabaseUpgradeTags` if called from `OnUpgradePerDatabase` — never both (use distinct tags).
- Reuse existing subscribers; max 2 nesting levels; upgrade tags only in upgrade code.

## 4. No outside calls

- Forbidden during upgrade: `HttpClient`, web services, DotNet interop, any external communication (failures block the upgrade and may not be rollback-able).

## 5. Execution context

- `if GetExecutionContext() = ExecutionContext::Upgrade then exit;` is allowed to skip code (e.g. report selections), but requires an explanatory comment and sparing use.

## 6. DataTransfer

- **Use `DataTransfer`** for tables that may exceed ~300,000 records, and when initializing fields/tables added in the same PR — never a `FindSet`/`Modify` loop.
- Use it **only** for new fields/tables. If the target is an existing field, add a comment noting that validation triggers and event subscribers will not fire.

```al
MyTableDataTransfer.SetTables(Database::"My Table", Database::"My Table");
MyTableDataTransfer.AddSourceFilter(MyTable.FieldNo(Status), '=%1', Status::Open);
MyTableDataTransfer.AddConstantValue(true, MyTable.FieldNo("New Field"));
// or AddFieldValue(SourceFieldNo, TargetFieldNo) to copy a field
MyTableDataTransfer.CopyFields();
Clear(MyTableDataTransfer); // required before reuse
```

## 7. InitValue

- `InitValue` applies only to new records; existing rows keep the datatype default. Every new field with `InitValue` needs upgrade code (preferably `DataTransfer.AddConstantValue`).

## Review checklist

1. Only procedure calls in `OnUpgrade*` triggers.
2. No `OnValidate`/`OnCheckPreconditions` upgrade triggers without justification.
3. All reads protected with `if ... then`.
4. Upgrade tags instead of version checks, and every tag registered in the matching subscriber.
5. No external calls.
6. `DataTransfer` for large tables, and only for new fields/tables.
7. Upgrade code present for every new `InitValue` field.
8. Minimal blocking error handling.
