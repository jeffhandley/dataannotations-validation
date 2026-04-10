# Chapter 7: ValidationContext Deep Dive

---
[<<-- Previous: Advanced Custom Validation](06-advanced-custom-validation.md) | [Table of Contents](README.md) | [Next: The ValidationResult API -->>\](08-validation-result-api.md)
---

`ValidationContext` is the bridge between the validation system and the application's runtime environment. Introduced in .NET 4.0 as part of RIA Services, it provides validators with access to the object being validated, service resolution, and arbitrary state — enabling rich, context-aware validation logic.

**Key References:**

- [Providing ValidationContext](https://jeffhandley.com/2010-10-25/riaservicesvalidationcontext) (Jeff Handley)
- [Using ValidationContext (Cross-Entity Validation)](https://jeffhandley.com/2010-10-25/crossentityvalidation) (Jeff Handley)
- [ValidationContext API](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationcontext)
- Source: [ValidationContext.cs](https://github.com/dotnet/runtime/blob/main/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations/ValidationContext.cs)

## Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `ObjectInstance` | `object` | The object being validated |
| `ObjectType` | `Type` | Type of the object |
| `MemberName` | `string?` | Property name being validated (null for object-level) |
| `DisplayName` | `string` | User-friendly name from `[Display]` or property name |
| `Items` | `IDictionary<object, object>` | Arbitrary state dictionary |

## IServiceProvider Implementation

`ValidationContext` implements `System.IServiceProvider`, enabling dependency injection within validators. This means you can resolve services during validation without coupling your validation logic to concrete implementations.

```csharp
var services = new ServiceCollection()
    .AddSingleton<IMeetingRepository>(repo)
    .BuildServiceProvider();

var context = new ValidationContext(meeting, services, items: null);
```

Inside a validator, you resolve services through the `ValidationContext`:

```csharp
protected override ValidationResult? IsValid(
    object? value, ValidationContext validationContext)
{
    var repo = validationContext.GetService(typeof(IMeetingRepository))
        as IMeetingRepository;

    if (repo is not null)
    {
        // Use the repository to check for conflicts
    }

    return ValidationResult.Success;
}
```

As Jeff Handley wrote:

> "Consider the scenario where a validator needs to access the database; you certainly wouldn't couple a validation method to your data access layer, would you? Instead, you would make your repository available as a service that your validation method can consume in a loosely-coupled manner."

## Items Dictionary

The `Items` property provides an arbitrary property bag for passing ad-hoc state into validators without needing a formal service registration. This is useful for contextual flags, the current user, or other per-invocation data.

```csharp
var items = new Dictionary<object, object>
{
    { "AllowOverBooking", false },
    { "CurrentUser", currentUser }
};

var context = new ValidationContext(meeting, serviceProvider: null, items: items);
```

Inside a validator, you can read from the `Items` dictionary:

```csharp
if (validationContext.Items.TryGetValue("AllowOverBooking", out var value)
    && value is bool allowOverBooking && allowOverBooking)
{
    return ValidationResult.Success;
}
```

## Cross-Field Validation Using ObjectInstance

Validators can inspect other properties on the object being validated by casting `ObjectInstance` to the expected type. However, there is a key subtlety described by Jeff Handley: **for the property being validated, use the `value` parameter since the property on the object still holds the old value.**

Here is a `NoTimeTravel` validator that ensures a meeting's start time is before its end time:

```csharp
public static ValidationResult? NoTimeTravel(
    DateTime time, ValidationContext validationContext)
{
    var meeting = (Meeting)validationContext.ObjectInstance;

    DateTime start = validationContext.MemberName == "Start" ? time : meeting.Start;
    DateTime end = validationContext.MemberName == "End" ? time : meeting.End;

    if (start > end)
    {
        return new ValidationResult(
            "Meetings cannot result in time travel.",
            new[] { "Start", "End" });
    }

    return ValidationResult.Success;
}
```

This example demonstrates three key techniques:

1. **Strongly-typed value parameter** — the `time` parameter gives direct access to the value being validated without casting from `object`.
2. **Conditional value extraction based on `MemberName`** — since the property on `ObjectInstance` still holds the old value, the validator checks which property is being validated and uses the `value` parameter for that property while reading the other property from `ObjectInstance`.
3. **Both member names in the result** — including both `"Start"` and `"End"` in the `MemberNames` collection ensures both fields highlight in the UI and changes to either field will clear the error.

## Cross-Entity Validation Using Services

For validation that spans beyond the object being validated — such as checking against other entities in the system — the `IServiceProvider` pattern shines. Here is the full pattern from Jeff Handley's blog:

```csharp
public interface IMeetingDataProvider
{
    IEnumerable<Meeting> Meetings { get; }
    IEnumerable<Location> Locations { get; }
}

public static ValidationResult? IsValidLocation(
    string location, ValidationContext validationContext)
{
    if (!string.IsNullOrEmpty(location))
    {
        var meetingData = validationContext.GetService(typeof(IMeetingDataProvider))
            as IMeetingDataProvider;

        if (meetingData is not null
            && !meetingData.Locations.Any(l => l.LocationName == location))
        {
            return new ValidationResult(
                "That is not a valid location",
                new[] { validationContext.MemberName! });
        }
    }

    return ValidationResult.Success;
}
```

This approach keeps the validation logic decoupled from the data access layer. The validator declares a dependency on `IMeetingDataProvider` and resolves it at validation time, allowing different implementations to be supplied in different contexts (e.g., client-side vs. server-side).

## Constructor Details

```csharp
// Full constructor
public ValidationContext(
    object instance,
    IServiceProvider? serviceProvider,
    IDictionary<object, object>? items)

// Shorthand
public ValidationContext(object instance)
```

When providing a seed `IServiceProvider`, calls to `GetService` are delegated to it. The `Items` dictionary is seeded from the provided dictionary (if any) and can be added to after construction — validators can both read and write to it during the validation process.

---
[<<-- Previous: Advanced Custom Validation](06-advanced-custom-validation.md) | [Table of Contents](README.md) | [Next: The ValidationResult API -->>\](08-validation-result-api.md)
---
