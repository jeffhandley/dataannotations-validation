# Chapter 2: Creating Custom Validation Attributes

<nav>

<a href="01-built-in-validation-attributes.md">← Previous: Built-In Validation Attributes</a> | <a href="README.md">Table of Contents</a> | <a href="03-annotating-objects.md">Next: Annotating Objects for Validation →</a>

</nav>

> **Key References**
>
> - [Custom Validation Methods](https://jeffhandley.com/2010-09-26/riaservicescustomvalidationmethods) (Jeff Handley)
> - [Custom Reusable Validators](https://jeffhandley.com/2010-09-26/riaservicescustomreusablevalidators) (Jeff Handley)
> - [ValidationAttribute API](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute)

When the built-in attributes don't cover your business rules, you have two approaches for creating custom validators: deriving from `ValidationAttribute` for reusable validators, or using `[CustomValidation]` for one-off business logic.

## Approach 1: Deriving from ValidationAttribute

This is the recommended approach for reusable validators that you'll apply across many models. Create a new class that inherits from `ValidationAttribute` and override the `IsValid` method that accepts a `ValidationContext`.

```csharp
public class FutureDateAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(
        object? value, ValidationContext validationContext)
    {
        if (value is DateTime date && date <= DateTime.Now)
        {
            return new ValidationResult(
                $"{validationContext.DisplayName} must be in the future.",
                new[] { validationContext.MemberName! });
        }

        return ValidationResult.Success;
    }
}
```

### The Dual-Overload Pattern

`ValidationAttribute` has two `IsValid` overloads:

1. **`public virtual bool IsValid(object? value)`** — The legacy overload from .NET 3.5. It receives only the raw value with no context.
2. **`protected virtual ValidationResult? IsValid(object? value, ValidationContext validationContext)`** — The modern overload from .NET 4.0+. It provides access to the `ValidationContext`, enabling richer error reporting and cross-property validation.

The base class uses an internal `_hasBaseIsValid` flag to detect which overload has been overridden, preventing infinite recursion between the two. **Always override option 2** — it gives you access to the validation context and lets you return detailed `ValidationResult` objects with member names.

> *"For a fun programming challenge — Figure out how ValidationAttribute can determine which version of IsValid has been overridden and should therefore be called during validation."*
>
> — Jeff Handley, [Custom Reusable Validators](https://jeffhandley.com/2010-09-26/riaservicescustomreusablevalidators)

## Approach 2: Using CustomValidationAttribute

For one-off business logic that's closely tied to a specific model, `[CustomValidation]` lets you point to a static method without creating a new attribute class.

```csharp
public static class MeetingValidators
{
    public static ValidationResult? NoEarlyMeetings(
        DateTime meetingStartTime,
        ValidationContext validationContext)
    {
        if (meetingStartTime.TimeOfDay.Hours < 9)
        {
            return new ValidationResult(
                "Meetings cannot be scheduled before 9:00 AM.",
                new[] { validationContext.MemberName! });
        }

        return ValidationResult.Success;
    }
}

// Applied as:
// [CustomValidation(typeof(MeetingValidators), "NoEarlyMeetings")]
```

## 10 Rules for Custom Validation Methods

From Jeff Handley's [Custom Validation Methods](https://jeffhandley.com/2010-09-26/riaservicescustomvalidationmethods):

1. The method's class must be public.
2. The class can be static or instance (but the method itself must be static).
3. The method must be public.
4. The method must be static.
5. The return type must be `ValidationResult`.
6. The first parameter accepts the value being validated (can be strongly-typed).
7. An optional second parameter of type `ValidationContext` is recommended.
8. No other parameters are allowed.
9. Return `ValidationResult.Success` on success.
10. Return `new ValidationResult(errorMessage, memberNames)` on failure.

## When to Use Each Approach

| Approach | Best For |
|----------|----------|
| **Deriving from `ValidationAttribute`** | Reusable validators applied across many models — date ranges, format checks, domain-specific constraints. These become part of your validation library. |
| **`[CustomValidation]`** | One-off business logic tied to a specific model — cross-property checks, rules that reference external context, or validation that doesn't warrant its own attribute class. |

<nav>

<a href="01-built-in-validation-attributes.md">← Previous: Built-In Validation Attributes</a> | <a href="README.md">Table of Contents</a> | <a href="03-annotating-objects.md">Next: Annotating Objects for Validation →</a>

</nav>
