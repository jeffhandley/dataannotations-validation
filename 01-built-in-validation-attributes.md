# Chapter 1: Built-In Validation Attributes

---
[<<-- Previous: Suggested Reading Order](00-suggested-reading-order.md) | [Table of Contents](README.md) | [Next: Creating Custom Validation Attributes -->>\](02-creating-custom-attributes.md)
---

> **Key References**
>
> - [System.ComponentModel.DataAnnotations Namespace](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations?view=net-9.0)
> - [Standard Validators](https://jeffhandley.com/2010-09-22/riaservicesstandardvalidators) (Jeff Handley)
> - Source: [dotnet/runtime DataAnnotations](https://github.com/dotnet/runtime/tree/main/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations)

.NET ships a rich set of built-in validation attributes, all deriving from `ValidationAttribute`. These attributes declaratively express constraints on properties and fields, and the `Validator` class evaluates them at runtime. Understanding each attribute's behavior — especially around null handling — is essential before building custom validators or integrating with frameworks like ASP.NET Core or Blazor.

## Core Validators (Since .NET 3.5 / 4.0)

| Attribute | Purpose | Key Behavior |
|-----------|---------|--------------|
| `[Required]` | Value must be non-null (optionally non-empty for strings) | `AllowEmptyStrings` (default false); always evaluated **first** by Validator |
| `[Range(min, max)]` | Numeric or IComparable range check | Supports int, double, arbitrary Type; `MinimumIsExclusive`/`MaximumIsExclusive` (.NET 8) |
| `[StringLength(max)]` | String max (optional min) length | `MinimumLength` property; returns Success for null |
| `[RegularExpression(pattern)]` | Regex match | `MatchTimeoutInMilliseconds`; returns Success for null/empty |
| `[Compare("OtherProperty")]` | Two-property equality | Sets `RequiresValidationContext = true`; uses reflection |

## Data-Type Validators

| Attribute | Purpose | Key Behavior |
|-----------|---------|--------------|
| `[EmailAddress]` | Simple email format (one @) | Extends DataTypeAttribute |
| `[Phone]` | Phone number | Digit counting with extension support |
| `[Url]` | URL validation | http://, https://, ftp:// schemes |
| `[CreditCard]` | Luhn algorithm | Strips dashes/spaces |
| `[FileExtensions]` | File extension whitelist | Default: "png,jpg,jpeg,gif" |
| `[EnumDataType(typeof(T))]` | Enum value check | Verifies value is defined in enum |

## Length/Collection Validators

| Attribute | Purpose | Key Behavior |
|-----------|---------|--------------|
| `[MaxLength(n)]` | Max length for strings or collections | ICollection.Count or Array.Length |
| `[MinLength(n)]` | Min length | Same collection support |
| `[Length(min, max)]` | Combined min+max (.NET 8) | Single attribute for both bounds |

## Value Constraint Validators (.NET 8+)

| Attribute | Purpose | Key Behavior |
|-----------|---------|--------------|
| `[AllowedValues(v1, v2, ...)]` | Whitelist | object[] params; supports null |
| `[DeniedValues(v1, v2, ...)]` | Blacklist | Mirror of AllowedValues |
| `[Base64String]` | Base64 format | .NET 8+ |

## Key Behaviors All Validators Share

> **"Nothing is Required, Except for Required"** — Jeff Handley, [Standard Validators](https://jeffhandley.com/2010-09-22/riaservicesstandardvalidators)

This is one of the most important design rules in DataAnnotations: validators like `[Range]`, `[StringLength]`, and `[RegularExpression]` all return **success** for blank or null values. Only `[Required]` rejects nulls and (by default) empty strings. This means that if a property is optional, those validators will silently pass when no value is provided — which is by design. If you want a value to be mandatory *and* constrained, you must stack `[Required]` alongside the other attribute.

### Error Messages and Placeholders

Every validation attribute supports customizable error messages:

- **`ErrorMessage`** — Inline format string with `{0}` as a placeholder for the display name of the property being validated.
- **`ErrorMessageResourceType`** / **`ErrorMessageResourceName`** — For localization, point to a resource file's type and key.

Different validators expose additional numbered placeholders beyond `{0}`:

1. **`[Required]`** — No additional placeholders.
2. **`[Range(min, max)]`** — `{1}` = minimum value, `{2}` = maximum value.
3. **`[StringLength(max)]`** — `{1}` = maximum length.
4. **`[StringLength(max, MinimumLength = min)]`** — `{1}` = maximum length, `{2}` = minimum length (note: `{2}` is the *minimum*, not the other way around!).
5. **`[RegularExpression(pattern)]`** — `{1}` = the regex pattern.

---
[<<-- Previous: Suggested Reading Order](00-suggested-reading-order.md) | [Table of Contents](README.md) | [Next: Creating Custom Validation Attributes -->>\](02-creating-custom-attributes.md)
---
