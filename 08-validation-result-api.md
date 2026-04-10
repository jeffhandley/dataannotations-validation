# Chapter 8: The ValidationResult API

---
[<<-- Previous: ValidationContext Deep Dive](07-validation-context.md) | [Table of Contents](README.md) | [Next: The Async Validation Gap -->>\](09-async-validation-gap.md)
---

`ValidationResult` is the currency of the DataAnnotations validation system — every validation operation ultimately produces either a `null` (success) or a `ValidationResult` instance (failure). Understanding its design, especially the `Success` sentinel, is essential for writing correct validators.

**Key References:**

- [ValidationResult API](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationresult)
- Source: [ValidationResult.cs](https://github.com/dotnet/runtime/blob/main/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations/ValidationResult.cs)

## The Success Sentinel

```csharp
public static readonly ValidationResult? Success; // initialized to null
```

**`ValidationResult.Success` is `null`.** This is a critical detail:

- Validators return `null` to indicate success.
- `Success` is a static readonly field, not a singleton object.
- Consumers compare `result == ValidationResult.Success` (i.e., `result == null`).

> **⚠️ Important:** Don't create a `new ValidationResult(null)` thinking it means success — that creates an error result with a null message. Always return `ValidationResult.Success` (which is `null`).

## Constructors

```csharp
public ValidationResult(string? errorMessage)
public ValidationResult(string? errorMessage, IEnumerable<string>? memberNames)
protected ValidationResult(ValidationResult validationResult) // copy constructor
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `ErrorMessage` | `string?` | The validation error message |
| `MemberNames` | `IEnumerable<string>` | Properties that caused the error (can be empty) |

## MemberNames Best Practices

From the blog series, there are three patterns for populating `MemberNames`:

**Single field — include the member name for UI highlighting:**

```csharp
new ValidationResult(
    "Invalid email.",
    new[] { validationContext.MemberName! })
```

**Cross-field — include all related members:**

```csharp
new ValidationResult(
    "Start must be before End.",
    new[] { "Start", "End" })
```

**Entity-level — omit if no clear field guidance:**

```csharp
new ValidationResult("Meeting is too expensive.")
```

When to use each pattern:

- **Property-level:** Always include `MemberName` so the UI highlights the correct field.
- **Cross-field:** Include all involved members so that changes to any of them clear the error.
- **Entity-level:** Omit member names if the user must decide which field(s) to change.

## ValidationException

Source: [ValidationException.cs](https://github.com/dotnet/runtime/blob/main/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations/ValidationException.cs)

`ValidationException` is thrown by `Validator.ValidateObject()` and `Validator.ValidateProperty()` when validation fails. It carries full context about the failure:

| Property | Type | Description |
|----------|------|-------------|
| `ValidationResult` | `ValidationResult` | The result that triggered the exception |
| `ValidationAttribute` | `ValidationAttribute?` | The attribute that failed |
| `Value` | `object?` | The value being validated |

```csharp
try
{
    Validator.ValidateObject(meeting, context, validateAllProperties: true);
}
catch (ValidationException ex)
{
    Console.WriteLine($"Error: {ex.ValidationResult.ErrorMessage}");
    Console.WriteLine($"Attribute: {ex.ValidationAttribute?.GetType().Name}");
    Console.WriteLine($"Value: {ex.Value}");
}
```

Note: `ValidateObject` throws on the **first** error, while `TryValidateObject` collects **all** errors.

## The Relationship Between ValidationResult and ValidationAttribute

Understanding how `ValidationResult` flows through the validation pipeline clarifies the responsibilities of each component:

1. `Validator` calls `ValidationAttribute.GetValidationResult(value, context)`.
2. `GetValidationResult` calls `IsValid(value, context)` internally.
3. If `IsValid` returns non-null (error), `GetValidationResult` formats the error message.
4. The formatted `ValidationResult` is returned (or `null` for success).

The key insight: `GetValidationResult` is responsible for calling `FormatErrorMessage`, which handles the `{0}` placeholders in error message templates. Custom validators that override `IsValid` don't need to call `FormatErrorMessage` themselves if they set `ErrorMessage` on the attribute — the pipeline handles it automatically.

---
[<<-- Previous: ValidationContext Deep Dive](07-validation-context.md) | [Table of Contents](README.md) | [Next: The Async Validation Gap -->>\](09-async-validation-gap.md)
---
