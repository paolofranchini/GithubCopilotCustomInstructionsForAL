---
description: AL Error handling patterns, debugging techniques, and troubleshooting guidelines for AL development
applyTo: "*.al"
---

# AL Error Handling & Troubleshooting Rules

Robust error handling and effective troubleshooting practices are essential for maintaining reliable Business Central applications.

## Rule 1: Use TryFunctions for Error Handling

### Intent
Implement proper error handling using TryFunctions to manage exceptions gracefully and provide meaningful user feedback. Use TryFunctions for error handling in scenarios where rollback is required, implement proper exception handling for external service calls, provide meaningful error messages to users, and log errors appropriately for debugging purposes. When generating code that might fail (external calls, data operations, calculations), implement appropriate TryFunction error handling and provide clear error messages.

### Examples

```al
// Good example - TryFunction with proper error handling and error labels
procedure ProcessPayment(Amount: Decimal): Boolean
var
  PaymentService: Codeunit "Payment Service";
  ErrorText: Text;
  PaymentProcessingFailedLbl: Label 'Payment processing failed: %1', Comment = '%1 = Error message';
  PaymentProcessingFailedTelemetryLbl: Label 'Payment processing failed', Locked = true;
begin
  if not TryProcessPaymentInternal(Amount) then begin
    ErrorText := GetLastErrorText();
    LogError(PaymentProcessingFailedTelemetryLbl, ErrorText);
    Message(PaymentProcessingFailedLbl, ErrorText);
    exit(false);
  end;
  
  exit(true);
end;

[TryFunction]
local procedure TryProcessPaymentInternal(Amount: Decimal)
var
  PaymentService: Codeunit "Payment Service";
begin
  PaymentService.ProcessPayment(Amount);
end;
```

```al
// Bad example (avoid hardcoded error messages and unhandled errors)
procedure ProcessPayment(Amount: Decimal)
var
  PaymentService: Codeunit "Payment Service";
begin
  // No error handling - will cause unhandled exceptions
  // Also avoid hardcoded messages like this:
  // Message('Payment could not be processed');
  PaymentService.ProcessPayment(Amount);
end;
```

## Rule 2: Use Error Labels for All Messages

### Intent
All error messages, warnings, and user messages must use label variables instead of hardcoded text. This ensures proper localization support and maintainability. Define labels with appropriate comments for translators and use Locked = true for technical messages that should not be translated.

### Examples

```al
// Good example - Using error labels
procedure ValidateBusinessLogic(SalesHeader: Record "Sales Header")
var
  Customer: Record Customer;
  CustomerNotFoundErr: Label 'Customer %1 does not exist for sales document %2.', Comment = '%1 = Customer No., %2 = Sales Header No.';
  CustomerBlockedErr: Label 'Customer %1 is blocked (%2). Cannot process sales document %3.', Comment = '%1 = Customer No., %2 = Blocked reason, %3 = Sales Header No.';
  EmptyHeaderNoErr: Label 'Sales header number cannot be empty.';
begin
  if SalesHeader."No." = '' then
    Error(EmptyHeaderNoErr);
  
  if not Customer.Get(SalesHeader."Sell-to Customer No.") then
    Error(CustomerNotFoundErr, SalesHeader."Sell-to Customer No.", SalesHeader."No.");
          
  if Customer.Blocked <> Customer.Blocked::" " then
    Error(CustomerBlockedErr, Customer."No.", Customer.Blocked, SalesHeader."No.");
end;
```

```al
// Bad example (avoid hardcoded error messages)
procedure ValidateBusinessLogic(SalesHeader: Record "Sales Header")
var
  Customer: Record Customer;
begin
  if not Customer.Get(SalesHeader."Sell-to Customer No.") then
    Error('Customer not found'); // Hardcoded - avoid this
    
  if Customer.Blocked <> Customer.Blocked::" " then
    Error('Customer blocked'); // Hardcoded - avoid this
end;
```

## Rule 3: Code Compilation and Correctness Priority

### Intent
Generated AL code should prioritize correctness over immediate compilation. Code can fail to compile if AI suggests base functions or events that don't exist, or if variables in event subscriptions are incorrect. When this happens, leave space for manual fixes rather than changing the intended behavior. If you're confident the logic should work as suggested but there are naming or parameter issues, leave it for user correction rather than altering the business logic.

### Examples

```al
// Good example - Correct logic even if function names need verification
procedure HandleCustomerModification(var Customer: Record Customer)
var
  CustomerValidation: Codeunit "Customer Validation"; // May need verification
begin
  // Correct business logic - even if codeunit name needs adjustment
  if not CustomerValidation.ValidateCustomerData(Customer) then
    Error(ValidationFailedErr);

  Customer.Modify(true);
end;
```

```al
// Good example - Event subscription with correct intent
[EventSubscriber(ObjectType::Table, Database::Customer, OnAfterModifyEvent, '', false, false)]
local procedure OnAfterCustomerModify(var Rec: Record Customer; var xRec: Record Customer; RunTrigger: Boolean)
var
  CustomerChangeLog: Codeunit "Customer Change Log"; // Function may need verification
begin
  // Correct logic - even if codeunit or method names need adjustment
  CustomerChangeLog.LogCustomerChange(Rec, xRec);
end;
```

## Rule 4: Custom Telemetry Implementation

### Intent
Add custom telemetry for tracking business-critical operations, but only when explicitly requested by the user. Use Session.LogMessage for custom telemetry with appropriate verbosity levels and data classification. Include relevant custom dimensions for context and use proper telemetry scope for extension publishers.

### Examples

```al
// Good example - Custom telemetry (only when user explicitly requests it)
procedure PostSalesDocument(var SalesHeader: Record "Sales Header")
var
  TelemetryCustomDimensions: Dictionary of [Text, Text];
  SalesDocPostedMsg: Label 'Sales document posted successfully', Locked = true;
  SalesDocPostFailedMsg: Label 'Sales document posting failed', Locked = true;
begin
  // Add context for telemetry
  TelemetryCustomDimensions.Add('DocumentType', Format(SalesHeader."Document Type"));
  TelemetryCustomDimensions.Add('CustomerNo', SalesHeader."Sell-to Customer No.");
  
  if TryPostSalesDocument(SalesHeader) then begin
    // Log successful operation
    Session.LogMessage('SAL001', SalesDocPostedMsg, 
                      Verbosity::Normal, DataClassification::SystemMetadata,
                      TelemetryScope::ExtensionPublisher, TelemetryCustomDimensions);
  end else begin
    // Log failed operation with error details
    TelemetryCustomDimensions.Add('ErrorText', GetLastErrorText());
    Session.LogMessage('SAL002', SalesDocPostFailedMsg, 
                      Verbosity::Error, DataClassification::SystemMetadata,
                      TelemetryScope::ExtensionPublisher, TelemetryCustomDimensions);
  end;
end;
```

## Rule 5: Telemetry Conventions

### Intent
Follow consistent conventions for telemetry events so they are discoverable, actionable, and privacy-safe. These conventions mirror the patterns used in standard BC telemetry.

- **DataClassification must be `SystemMetadata`** — any other value causes the event to be silently suppressed by the BC server and never reach Application Insights. Never log customer or personal data.
- **EventId must be unique and prefixed** — use a short app-specific prefix plus a zero-padded number (e.g. `MyApp-0001`). EventIds are an API: changing them is a breaking change.
- **Message follows "Object ActionInPastTense" pattern** — e.g. `'Sales document posted'`, `'Payment processing failed'`. Include key dimension values in the message so events are readable without opening custom dimensions.
- **Custom dimension key names use PascalCase** — the BC server automatically prefixes them with `al` in Application Insights (e.g. key `DocumentType` appears as `alDocumentType`). Never use spaces in dimension key names.
- **Use `TelemetryScope::ExtensionPublisher`** to send only to your app's Application Insights resource. Use `TelemetryScope::All` only when the event is also relevant for the environment administrator.

### Examples

```al
// Good example - All conventions applied
procedure ProcessPayment(PaymentAmount: Decimal; CustomerNo: Code[20])
var
  Dimensions: Dictionary of [Text, Text];
  PaymentProcessedMsg: Label 'Payment processed', Locked = true;
  PaymentFailedMsg: Label 'Payment processing failed', Locked = true;
begin
  Dimensions.Add('CustomerNo', CustomerNo);
  Dimensions.Add('Amount', Format(PaymentAmount));

  if TryProcessPaymentInternal(PaymentAmount) then
    Session.LogMessage(
      'MyApp-0001',                          // unique, prefixed EventId
      PaymentProcessedMsg,                   // "Object ActionInPastTense"
      Verbosity::Normal,
      DataClassification::SystemMetadata,    // REQUIRED — other values suppress the event
      TelemetryScope::ExtensionPublisher,
      Dimensions)                            // PascalCase keys → alCustomerNo, alAmount in AI
  else begin
    Dimensions.Add('ErrorText', GetLastErrorText());
    Session.LogMessage(
      'MyApp-0002',
      PaymentFailedMsg,
      Verbosity::Error,
      DataClassification::SystemMetadata,
      TelemetryScope::ExtensionPublisher,
      Dimensions);
  end;
end;
```

```al
// Bad example - common mistakes
Session.LogMessage(
  'Error',                              // BAD: not unique, no prefix
  'Something went wrong for ' + CustomerName,  // BAD: embeds customer data
  Verbosity::Error,
  DataClassification::CustomerContent, // BAD: event is silently suppressed
  TelemetryScope::All,                 // BAD: unnecessarily broad scope
  'customer name', CustomerName);      // BAD: spaces in key, personal data in value
```