---
description: AL Error handling patterns, debugging techniques, and troubleshooting guidelines for AL development
applyTo: "*.al"
---

# AL Error Handling & Troubleshooting Rules

## Rule 1: TryFunctions

- Wrap failure-prone code (external calls, risky data operations) in a `[TryFunction]` local procedure.
- On failure read `GetLastErrorText()`, log it, and return a meaningful result to the caller.

```al
if not TryProcessPaymentInternal(Amount) then begin
    ErrorText := GetLastErrorText();
    Message(PaymentFailedLbl, ErrorText);
    exit(false);
end;
```

## Rule 2: Labels for every message

- Never hardcode text in `Error`, `Message`, `Confirm`, `StrMenu`.
- Declare `Label` variables with `Comment = '%1 = ...'` for placeholders; use `Locked = true` for technical/telemetry text.
- Suffix conventions: `...Err`, `...Msg`, `...Qst`, `...Lbl`, `...Tok`.

## Rule 3: Correctness over compilation

- Keep the intended business logic even if an object/procedure name or event signature needs manual verification; do not alter behaviour just to make code compile.

## Rule 4 & 5: Telemetry

Add `Session.LogMessage` telemetry only when the user explicitly asks for it, and follow these conventions:

- `DataClassification::SystemMetadata` is **mandatory** — any other value silently suppresses the event.
- Never log customer or personal data (neither in the message nor in dimensions).
- EventId: unique, app-prefixed, zero-padded (`MyApp-0001`); it is an API, changing it is breaking.
- Message: "Object ActionInPastTense" (`'Sales document posted'`), `Locked = true`.
- Custom dimension keys: PascalCase, no spaces (surfaced as `alDocumentType` in App Insights).
- Scope: `TelemetryScope::ExtensionPublisher`; use `All` only when relevant to the environment admin.
- Verbosity: `Normal` for success, `Error`/`Warning` for failures (add `ErrorText` dimension).

```al
Dimensions.Add('CustomerNo', CustomerNo);
Session.LogMessage('MyApp-0001', PaymentProcessedMsg, Verbosity::Normal,
    DataClassification::SystemMetadata, TelemetryScope::ExtensionPublisher, Dimensions);
```
