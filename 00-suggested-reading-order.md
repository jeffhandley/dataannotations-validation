---
[Table of Contents](README.md) | [Next: Built-In Validation Attributes -->>](01-built-in-validation-attributes.md)
---

# Suggested Reading Order

For team members who are new to DataAnnotations validation, this recommended reading order starts with the foundational blog series, moves through official API documentation, then application model integration, and finishes with async validation concepts.

## Phase 1: Historical Foundation (Jeff Handley's Blog Series)

DataAnnotations validation saw its most significant innovation during the .NET RIA Services era (2009–2012), when APIs like `IValidatableObject`, `ValidationResult`, `ValidationContext`, and `Validator` were designed and refined. Jeff Handley's 2010 blog series was written against RIA Services but covers the universal DataAnnotations framework, and it remains one of the most thorough walkthroughs available to get started.

1. [Standard Validators](https://jeffhandley.com/2010-09-22/riaservicesstandardvalidators) — `[Required]`, `[Range]`, `[StringLength]`, `[RegularExpression]`; error messages and localization
2. [Custom Validation Methods](https://jeffhandley.com/2010-09-26/riaservicescustomvalidationmethods) — Using `[CustomValidation]` to delegate to static methods
3. [Custom Reusable Validators](https://jeffhandley.com/2010-09-26/riaservicescustomreusablevalidators) — Deriving from `ValidationAttribute`; the `IsValid(object, ValidationContext)` override
4. [Attribute Propagation](https://jeffhandley.com/archive/2010/09/30/RiaServicesValidationAttributePropagation.aspx) — How validators flow between tiers
5. [Validation Triggers](https://jeffhandley.com/archive/2010/10/06/RiaServicesValidationTriggers.aspx) — When validation is invoked
6. [Cross-Field Validation](https://jeffhandley.com/2010-10-10/crossfieldvalidation) — Validating one property based on another
7. [Entity-Level Validation](https://jeffhandley.com/2010-10-12/entitylevelvalidation) — Class-level validation and the 4-stage pipeline
8. [Providing ValidationContext](https://jeffhandley.com/2010-10-25/riaservicesvalidationcontext) — Items dictionary, IServiceProvider, ServiceContainer
9. [Using ValidationContext (Cross-Entity Validation)](https://jeffhandley.com/2010-10-25/crossentityvalidation) — Service-based validators that access data stores
10. [ViewModel Validation with Entity Rules](https://jeffhandley.com/archive/2011/09/06/ViewModelValidation.aspx) — Reusing validation across view models

> **Additional Resources:**
>
> - GitHub repository with sample code: [jeffhandley/RIAServicesValidation](https://github.com/jeffhandley/RIAServicesValidation)
> - Video tutorials (VB): [RIA Services Validation Video Tutorials](https://jeffhandley.com/2010-11-20/riaservicesvalidationvideotutorials)

## Phase 2: Modern .NET API Documentation

11. [System.ComponentModel.DataAnnotations Namespace](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations?view=net-9.0)
12. [Validator Class API](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validator?view=net-9.0)
13. [ValidationAttribute Class](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute?view=net-9.0)
14. [ValidationContext Class](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationcontext?view=net-9.0)
15. [ValidationResult Class](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationresult?view=net-9.0)

## Phase 3: Application Model Integration

16. [ASP.NET Core MVC Model Validation](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation)
17. [Blazor Forms Validation](https://learn.microsoft.com/en-us/aspnet/core/blazor/forms/validation)
18. [Options Pattern Validation](https://learn.microsoft.com/en-us/dotnet/core/extensions/options) — `ValidateDataAnnotations()`, `ValidateOnStart()`
19. [EF Core Data Annotations](https://learn.microsoft.com/en-us/ef/core/modeling/entity-properties)
20. [ASP.NET Core OpenAPI](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi)
21. [WPF Data Validation](https://learn.microsoft.com/en-us/dotnet/desktop/wpf/data/)
22. [WinForms Input Validation](https://learn.microsoft.com/en-us/dotnet/desktop/winforms/input-keyboard/validation)

## Phase 4: Async Validation Concepts

23. [Strickland Documentation](https://strickland.io) / [GitHub](https://github.com/jeffhandley/strickland)
24. [Strickland: Inspiration](https://github.com/jeffhandley/strickland/blob/master/docs/inspiration.md)
25. [Strickland: Two-Stage Sync/Async Validation](https://github.com/jeffhandley/strickland/blob/master/docs/async-validation/two-stage-sync-async-validation.md)

## Phase 5: Integration History and Working Prototype

26. [Chapter 11: The History of DataAnnotations Integration Across .NET](11-integration-history.md) — Chronological history of every integration point with PR/issue citations
27. [Chapter 12: The Async Validation Prototype — A Working Demo](12-async-validation-demo.md) — Analysis of the [`oroztocil/validation-demo`](https://github.com/dotnet/aspnetcore/tree/oroztocil/validation-demo) branch
28. [Microsoft.Extensions.Validation design issue (dotnet/aspnetcore#46349)](https://github.com/dotnet/aspnetcore/issues/46349) — Unified validation design proposal
29. [Minimal API validation PR (dotnet/aspnetcore#60724)](https://github.com/dotnet/aspnetcore/pull/60724) — Core implementation of Minimal API validation

---
[Table of Contents](README.md) | [Next: Built-In Validation Attributes -->>](01-built-in-validation-attributes.md)
---
