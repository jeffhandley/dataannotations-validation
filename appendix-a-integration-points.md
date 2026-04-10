# Appendix A: .NET Integration Points Catalog

---
[← Previous: The Async Validation Prototype — A Working Demo](12-async-validation-demo.md) | [Table of Contents](README.md) | [Next: Appendix B: Complete Reference Link Library →](appendix-b-references.md)
---

Every location across the .NET product suite where DataAnnotations validation is invoked or respected. **Each is a location that must be updated to support async validation.**

```mermaid
graph TB
    subgraph "Tier 1: Core Library"
        Core["System.ComponentModel.Annotations\n(Validator, ValidationAttribute,\nIValidatableObject)"]
    end

    subgraph "Tier 2: Options"
        Options["Microsoft.Extensions.Options\n(DataAnnotationValidateOptions,\nSource Generator)"]
    end

    subgraph "Tier 3-4: Web Frameworks"
        MVC["ASP.NET Core MVC\n(DataAnnotationsModelValidator)"]
        Blazor["Blazor Forms\n(DataAnnotationsValidator)"]
    end

    subgraph "Tier 5-6: Schema & Data"
        OpenAPI["OpenAPI Schema\n(JsonNodeSchemaExtensions)"]
        EF["EF Core\n(Model Configuration)"]
    end

    subgraph "Tier 7-8: New & Desktop"
        ExtVal["Microsoft.Extensions.Validation\n(.NET 10+)"]
        WPF["WPF / WinForms\n(Manual / Adapter)"]
    end

    Core --> Options
    Core --> MVC
    Core --> Blazor
    Core --> OpenAPI
    Core --> EF
    Core --> ExtVal
    Core --> WPF
```

## Tier 1: Core Validation Library (dotnet/runtime)

| Component | Location | How It Invokes Validation |
|-----------|----------|--------------------------|
| **`Validator` class** | `System.ComponentModel.Annotations` | Central orchestrator: `TryValidateObject`, `ValidateObject`, `TryValidateProperty`, `ValidateProperty`, `TryValidateValue`, `ValidateValue` |
| **`ValidationAttribute.GetValidationResult`** | `System.ComponentModel.Annotations` | Called by Validator to execute individual attribute validation |
| **`IValidatableObject.Validate`** | `System.ComponentModel.Annotations` | Called by Validator as the final validation stage |

## Tier 2: Microsoft.Extensions.Options (dotnet/runtime)

| Component | Location | How It Invokes Validation |
|-----------|----------|--------------------------|
| **`DataAnnotationValidateOptions<T>`** | `Microsoft.Extensions.Options.DataAnnotations` | Calls `Validator.TryValidateObject(...)` on options instances; recursively validates nested members |
| **`OptionsBuilderDataAnnotationsExtensions`** | `Microsoft.Extensions.Options.DataAnnotations` | `.ValidateDataAnnotations()` registers `DataAnnotationValidateOptions<T>` |
| **Options Source Generator** | `Microsoft.Extensions.Options` (gen) | Emits `Validator.TryValidateValue(...)` and `IValidatableObject.Validate(...)` in generated code |
| **`ValidateOptionsResultBuilder`** | `Microsoft.Extensions.Options` | Consumes `ValidationResult` objects, converts to `ValidateOptionsResult` |
| **`ValidateOnStart`** | `Microsoft.Extensions.Options` | Triggers validation at host startup |

## Tier 3: ASP.NET Core MVC (dotnet/aspnetcore)

| Component | Location | How It Invokes Validation |
|-----------|----------|--------------------------|
| **`DataAnnotationsModelValidator`** | `Microsoft.AspNetCore.Mvc.DataAnnotations` | Calls `Attribute.GetValidationResult(value, context)` per attribute — does NOT use `Validator.TryValidateObject` |
| **`DataAnnotationsModelValidatorProvider`** | `Microsoft.AspNetCore.Mvc.DataAnnotations` | Creates validator instances for attributes on model properties |
| **`ValidatableObjectAdapter`** | `Microsoft.AspNetCore.Mvc.DataAnnotations` | Invokes `IValidatableObject.Validate(context)` |
| **Model Binding Pipeline** | `Microsoft.AspNetCore.Mvc.Core` | Orchestrates validators during binding, populates ModelState |
| **`[ApiController]` auto-validation** | `Microsoft.AspNetCore.Mvc.Core` | Returns 400 when ModelState.IsValid is false |
| **`[Remote]` attribute** | `Microsoft.AspNetCore.Mvc.ViewFeatures` | AJAX property validation (browser-only) |

## Tier 4: Blazor (dotnet/aspnetcore)

| Component | Location | How It Invokes Validation |
|-----------|----------|--------------------------|
| **`DataAnnotationsValidator`** | `Microsoft.AspNetCore.Components.Forms` | Attaches to EditContext |
| **`EditContextDataAnnotationsExtensions`** | `Microsoft.AspNetCore.Components.Forms` | Field changes: `Validator.TryValidateProperty(...)`; form submit: `Validator.TryValidateObject(...)` |
| **`EditContext` / `ValidationMessageStore`** | `Microsoft.AspNetCore.Components.Forms` | Manages validation state, displays messages |

## Tier 5: OpenAPI / Swagger (dotnet/aspnetcore)

| Component | Location | How It Uses Validation Attributes |
|-----------|----------|-----------------------------------|
| **`JsonNodeSchemaExtensions`** | `Microsoft.AspNetCore.OpenApi` | Maps `[Range]` → min/max, `[MaxLength]`/`[MinLength]` → maxLength/minLength, `[StringLength]` → both, `[Required]` → required |
| **Swashbuckle** | Third-party NuGet | Similarly reads DataAnnotations for schema |
| **NSwag** | Third-party NuGet | Similarly reads DataAnnotations for schema |

## Tier 6: Entity Framework Core (dotnet/efcore)

| Component | How It Uses Validation Attributes |
|-----------|-----------------------------------|
| **Model configuration** | Reads `[Required]`, `[MaxLength]`, `[StringLength]`, `[Key]`, etc. for schema generation and migration |
| **Runtime validation** | EF Core does **NOT** run general DataAnnotations validation on save; uses attributes for model/schema configuration only |

## Tier 7: Microsoft.Extensions.Validation (.NET 10+)

| Component | Location | How It Invokes Validation |
|-----------|----------|--------------------------|
| **`AddValidation()` extension** | `Microsoft.Extensions.Validation` | Unified validation registration for DI; available outside HTTP scenarios |
| **Validation infrastructure** | `Microsoft.Extensions.Validation` | New .NET 10 package — APIs moved from ASP.NET Core |

## Tier 8: WPF (dotnet/wpf)

| Component | How It Uses Validation |
|-----------|----------------------|
| **Data Binding** | Uses `ValidationRule`, `IDataErrorInfo`, `INotifyDataErrorInfo` — not DataAnnotations natively |
| **Community adapters** | Various libraries bridge DataAnnotations to WPF binding validation |

## Tier 9: WinForms (dotnet/winforms)

| Component | How It Uses Validation |
|-----------|----------------------|
| **`Validating` events / `ErrorProvider`** | Event-based; DataAnnotations not the native mechanism |
| **Manual integration** | Requires explicit `Validator.TryValidateObject()` calls |

## Tier 10: .NET Aspire / Configuration

| Component | How It Uses Validation |
|-----------|----------------------|
| **`ValidateDataAnnotations()` + `ValidateOnStart()`** | Options pattern using `DataAnnotationValidateOptions<T>` |
| **Aspire service defaults** | Common pattern: `builder.Services.AddOptions<T>().ValidateDataAnnotations().ValidateOnStart()` |

## Tier 11: Minimal APIs

| Component | How It Uses Validation |
|-----------|----------------------|
| **No built-in validation** | Minimal APIs do NOT have automatic model validation |
| **Manual or library-based** | Developers call `Validator.TryValidateObject()` manually |

## Summary: Invocation Paths That Need Async Support

1. **`Validator` class** — Core static methods (dotnet/runtime)
2. **`ValidationAttribute.IsValid` / `GetValidationResult`** — Individual attribute execution (dotnet/runtime)
3. **`IValidatableObject.Validate`** — Object self-validation (dotnet/runtime)
4. **`DataAnnotationValidateOptions<T>`** — Options validation (dotnet/runtime)
5. **Options Source Generator** — Generated validation code (dotnet/runtime)
6. **`DataAnnotationsModelValidator`** — MVC model validation (dotnet/aspnetcore)
7. **`ValidatableObjectAdapter`** — MVC IValidatableObject (dotnet/aspnetcore)
8. **`EditContextDataAnnotationsExtensions`** — Blazor form validation (dotnet/aspnetcore)
9. **`Microsoft.Extensions.Validation`** — .NET 10 unified validation (dotnet/aspnetcore)
10. **OpenAPI schema generation** — May need schema representation for async validators
11. **Any future Minimal API validation** — If built-in validation is added

---
[← Previous: The Async Validation Prototype — A Working Demo](12-async-validation-demo.md) | [Table of Contents](README.md) | [Next: Appendix B: Complete Reference Link Library →](appendix-b-references.md)
---
