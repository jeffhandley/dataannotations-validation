# Chapter 3: Annotating Objects for Validation

---
[<<-- Previous: Creating Custom Validation Attributes](02-creating-custom-attributes.md) | [Table of Contents](README.md) | [Next: Programmatic Validation -->>\](04-programmatic-validation.md)
---

> **Key Reference**
>
> - [Standard Validators](https://jeffhandley.com/2010-09-22/riaservicesstandardvalidators) (Jeff Handley)

With the built-in and custom validation attributes in hand, the next step is applying them to your model classes. This chapter covers property-level annotations, type-level annotations, display metadata, and how multiple attributes interact when stacked on a single member.

## Property-Level Annotations

Most validation attributes are applied to individual properties. Here's a complete `Meeting` entity showing how to stack attributes, use localized error messages, and combine built-in validators with custom ones:

```csharp
using System.ComponentModel.DataAnnotations;

public class Meeting
{
    [Key]
    public int MeetingId { get; set; }

    [Required]
    [CustomValidation(typeof(MeetingValidators), "NoEarlyMeetings")]
    public DateTime Start { get; set; }

    [Required]
    public DateTime End { get; set; }

    [Required]
    [StringLength(80, MinimumLength = 5,
        ErrorMessageResourceType = typeof(ValidationErrorResources),
        ErrorMessageResourceName = "TitleStringLengthErrorMessage")]
    public string Title { get; set; } = string.Empty;

    [Required]
    [RegularExpression(@"\d{1,3}/\d{4}",
        ErrorMessage = "{0} must be in the format of 'Building/Room'")]
    public string Location { get; set; } = string.Empty;

    [Range(2, 100)]
    [Display(Name = "Minimum Attendees")]
    public int MinimumAttendees { get; set; }

    [Range(2, 100)]
    [Display(Name = "Maximum Attendees")]
    public int MaximumAttendees { get; set; }
}
```

## Type-Level Annotations

Validation attributes can also be applied at the class level. This is especially useful for cross-property validation rules that don't belong to any single property. When the `Validator` evaluates type-level attributes, the `ValidationContext.MemberName` is null, indicating the validation applies to the object as a whole.

```csharp
[CustomValidation(typeof(MeetingValidators), "PreventExpensiveMeetings")]
public class Meeting
{
    // ...
}
```

## Display Metadata

The `[Display(Name = "...")]` attribute controls how property names appear in validation error messages. Without it, the raw property name is used — which often results in awkward, developer-facing text in user-visible messages.

For example, without `[Display]`, a `[Range]` error on `MinimumAttendees` would produce:

> "The field MinimumAttendees must be between 2 and 100."

With `[Display(Name = "Minimum Attendees")]`, it becomes:

> "The field Minimum Attendees must be between 2 and 100."

## Stacking Multiple Attributes

Multiple validation attributes can be stacked on a single property. They are evaluated in an undefined order — with one important exception: `[Required]` is always evaluated **first** by the `Validator`. If any attribute fails, the property is considered invalid.

Here's a more complex .NET 8+ example showing several attributes stacked together:

```csharp
public class UserProfile
{
    [Required(ErrorMessage = "Username is required")]
    [StringLength(20, MinimumLength = 3)]
    [RegularExpression(@"^[a-zA-Z0-9_]+$",
        ErrorMessage = "Username can only contain letters, numbers, and underscores")]
    public string Username { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Range(13, 120)]
    public int Age { get; set; }

    [AllowedValues("Admin", "User", "Guest")]
    public string Role { get; set; } = "User";
}
```

When `Validator.TryValidateObject` runs with `validateAllProperties: true`, it evaluates `[Required]` first for each property, then proceeds to the remaining attributes. If `[Required]` fails, the other attributes on that property are skipped. This prevents confusing cascading errors — there's no point reporting that a string doesn't match a regex if the string was never provided in the first place.

---
[<<-- Previous: Creating Custom Validation Attributes](02-creating-custom-attributes.md) | [Table of Contents](README.md) | [Next: Programmatic Validation -->>\](04-programmatic-validation.md)
---
