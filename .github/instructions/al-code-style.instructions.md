---
description: AL Code structure, formatting, and folder organization guidelines for AL development
applyTo: "*.al"
---

# AL Code Style & Formatting Rules

These rules ensure consistent code structure and organization across AL projects, making code more maintainable and AI-assistant friendly.

## Style guidelines for AL code
  - Always use PascalCase for variable and function names.
  - Use PascalCase for object names (e.g., tables, pages, reports).
  - Maintain a consistent indentation style (4 spaces, per Microsoft AL standard).

## Commonly used methods and patterns
  - Temporary tables for performance optimization
  - Use of events for extensibility

## Rule 1: Consistent Indentation and Formatting

### Intent
Maintain consistent code formatting to improve readability and enable better AI understanding of code structure. Use 4-space indentation throughout your project, consistent with the official Microsoft AL standard. Maintain consistent formatting within functions and procedures.

### Examples

```al
// Good example
procedure CalculateDiscount(Amount: Decimal; DiscountPct: Decimal): Decimal
begin
  if DiscountPct > 0 then
    exit(Amount * DiscountPct / 100);
    
  exit(0);
end;
```

## Rule 2: Feature-Based Folder Organization

### Intent
Organize code by business features rather than object types to improve maintainability and logical grouping. Use feature-based organization with `src/feature/subfeature/` structure and place shared components in `Common` or `Shared` folders.

### Examples

```
// Good example - Feature-based organization
src/
├── NoSeries/
│   ├── NoSeries.Table.al
│   ├── NoSeries.Page.al
│   └── NoSeriesSetup.Codeunit.al
├── Sales/
│   ├── Invoice/
│   │   ├── SalesInvoice.Page.al
│   │   └── SalesInvoicePosting.Codeunit.al
│   └── Order/
│       └── SalesOrder.Page.al
└── Common/
    ├── Helpers/
    │   └── DateHelper.Codeunit.al
    └── Interfaces/
        └── IPostable.Interface.al
```

```
// Bad example (avoid object-type segregation)
src/
├── Tables/
│   ├── NoSeries.Table.al
│   └── SalesHeader.Table.al
├── Pages/
│   ├── NoSeries.Page.al
│   └── SalesInvoice.Page.al
└── Codeunits/
    ├── NoSeriesSetup.Codeunit.al
    └── SalesInvoicePosting.Codeunit.al
```

## Rule 3: Code Documentation and Comments

### Intent
Provide clear documentation for global functions using XML documentation comments. Code should be self-documenting through clear naming, but global functions in codeunits require proper documentation for API clarity.

### Examples

```al
// Good example - XML documentation for global functions
codeunit 50100 "Base64 Convert"
{
    /// <summary>
    /// Converts the value of the input string to its equivalent string representation that is encoded with base-64 digits.
    /// </summary>
    /// <param name="String">The string to convert.</param>
    /// <returns>The string representation, in base-64, of the input string.</returns>
    procedure ToBase64(String: Text): Text
    begin
        exit(Base64ConvertImpl.ToBase64(String));
    end;

    /// <summary>
    /// Validates discount percentage against business rules.
    /// </summary>
    /// <param name="DiscountPct">The discount percentage to validate.</param>
    procedure ValidateDiscountPercentage(DiscountPct: Decimal)
    begin
        if DiscountPct > 50 then
            Error('Discount cannot exceed 50% due to company policy');
            
        if DiscountPct < 0 then
            Error('Discount percentage cannot be negative');
    end;
}
```

```al
// Bad example (avoid inline comments for obvious operations)
procedure ValidateDiscountPercentage(DiscountPct: Decimal)
begin
  // Check if discount is greater than 50
  if DiscountPct > 50 then
    Error('Discount cannot exceed 50%');
    
  // Check if discount is less than 0
  if DiscountPct < 0 then
    Error('Discount percentage cannot be negative');
end;
```

## Rule 4: Modular and Reusable Code Structure

### Intent
Keep code modular and reusable to enhance maintainability and reduce duplication. Write small, focused procedures that do one thing well and use interfaces and patterns where appropriate.

### Examples

```al
// Good example - Modular approach
procedure PostDocument(var DocumentHeader: Record "Sales Header")
begin
  ValidateDocument(DocumentHeader);
  CalculateTotals(DocumentHeader);
  CreateLedgerEntries(DocumentHeader);
  UpdateStatus(DocumentHeader);
end;

local procedure ValidateDocument(var DocumentHeader: Record "Sales Header")
begin
  if DocumentHeader."No." = '' then
    Error('Document number cannot be empty');
end;

local procedure CalculateTotals(var DocumentHeader: Record "Sales Header")
begin
  DocumentHeader.CalcFields(Amount);
end;
```

```al
// Bad example (avoid monolithic procedures)
procedure PostDocument(var DocumentHeader: Record "Sales Header")
begin
  // All validation, calculation, and posting logic in one procedure
  // ... 200+ lines of mixed concerns
end;
```

## Rule 5: AL Object Internal File Structure

### Intent
Every AL object file must follow the mandatory ordering defined by Microsoft. Violating this order reduces readability and can cause unexpected behaviour with tools like the AL Formatter.

The required sequence inside any AL object is:
1. **Properties** (e.g. `Subtype`, `TableType`, `Access`)
2. **Object-specific constructs** in this order:
   - Table fields
   - Page layout
   - Actions
   - Triggers
3. **Global variables**
   - Labels first
   - Other global variables
4. **Methods** (procedures and functions)

### Examples

```al
// Good example - Correct object structure order
codeunit 50100 "Sales Document Validator"
{
    // 1. Properties
    Access = Public;

    // 2. Triggers
    trigger OnRun()
    begin
        ValidateAll();
    end;

    // 3. Global variables — Labels first, then others
    var
        DocumentValidatedMsg: Label 'Document %1 validated.', Comment = '%1 = Document No.';
        ValidationFailedErr: Label 'Validation failed: %1', Comment = '%1 = reason';
        IsInitialized: Boolean;

    // 4. Methods
    procedure ValidateAll()
    begin
        // implementation
    end;

    local procedure Initialize()
    begin
        IsInitialized := true;
    end;
}
```

## Rule 6: UI String Casing Conventions

### Intent
Use consistent casing for all user-facing labels, captions, and tooltips to maintain a professional and native BC look.

- **Sentence case** when the string contains a verb or reads as a sentence: `'Calculate total'`, `'Post document'`, `'Turn index off'`
- **Title case** when the string is a noun phrase without a verb: `'Customer Card'`, `'General Ledger'`, `'Sales Order'`

### Examples

```al
// Good examples
action(PostDocument)
{
    Caption = 'Post document';          // verb present → sentence case
    ToolTip = 'Post the sales document.';
}

field("Customer Name"; Rec."Customer Name")
{
    Caption = 'Customer Name';          // noun phrase → title case
    ToolTip = 'Specifies the name of the customer.';
}

action(CalculateDiscount)
{
    Caption = 'Calculate discount';     // verb present → sentence case
}
```

```al
// Bad examples
action(PostDocument)
{
    Caption = 'Post Document';          // BAD: title case with verb
}

field("Customer Name"; Rec."Customer Name")
{
    Caption = 'customer name';          // BAD: all lowercase
}
```