# Suggested Reading Order

<nav>

<a href="README.md">Table of Contents</a> | <a href="01-built-in-validation-attributes.md">Next: Built-In Validation Attributes →</a>

</nav>

For team members who are new to DataAnnotations validation, this recommended reading order starts with the foundational blog series, moves through official API documentation, then application model integration, and finishes with async validation concepts.

## Phase 1: Historical Foundation (RIA Services Blog Series)

DataAnnotations validation saw its most significant innovation during the .NET RIA Services era (2009–2012), when APIs like `IValidatableObject`, `ValidationResult`, `ValidationContext`, and `Validator` were designed and refined. The 2010 RIA Services Validation blog series was written against RIA Services but covers the universal DataAnnotations framework, and it remains one of the most thorough walkthroughs available.

1. [Standard Validators][blog-standard-validators] — `[Required]`, `[Range]`, `[StringLength]`, `[RegularExpression]`; error messages and localization
2. [Custom Validation Methods][blog-custom-methods] — Using `[CustomValidation]` to delegate to static methods
3. [Custom Reusable Validators][blog-custom-reusable] — Deriving from `ValidationAttribute`; the `IsValid(object, ValidationContext)` override
4. [Attribute Propagation][blog-attribute-propagation] — How validators flow between tiers
5. [Validation Triggers][blog-validation-triggers] — When validation is invoked
6. [Cross-Field Validation][blog-cross-field] — Validating one property based on another
7. [Entity-Level Validation][blog-entity-level] — Class-level validation and the 4-stage pipeline
8. [Providing ValidationContext][blog-providing-context] — Items dictionary, IServiceProvider, ServiceContainer
9. [Using ValidationContext (Cross-Entity Validation)][blog-cross-entity] — Service-based validators that access data stores
10. [ViewModel Validation with Entity Rules][blog-viewmodel-validation] — Reusing validation across view models

> **Additional Resources:**
>
> - GitHub repository with sample code: [jeffhandley/RIAServicesValidation][github-ria-validation]
> - Video tutorials (VB): [RIA Services Validation Video Tutorials][blog-video-tutorials]

## Phase 2: Modern .NET API Documentation

11. [System.ComponentModel.DataAnnotations Namespace][api-namespace]
12. [Validator Class API][api-validator]
13. [ValidationAttribute Class][api-validation-attribute]
14. [ValidationContext Class][api-validation-context]
15. [ValidationResult Class][api-validation-result]

## Phase 3: Application Model Integration

16. [ASP.NET Core MVC Model Validation][docs-mvc-validation]
17. [Blazor Forms Validation][docs-blazor-validation]
18. [Options Pattern Validation][docs-options-validation] — `ValidateDataAnnotations()`, `ValidateOnStart()`
19. [EF Core Data Annotations][docs-ef-core]
20. [ASP.NET Core OpenAPI][docs-openapi]
21. [WPF Data Validation][docs-wpf-validation]
22. [WinForms Input Validation][docs-winforms-validation]

## Phase 4: Async Validation Concepts

23. [Strickland Documentation][strickland-docs] / [GitHub][strickland-github]
24. [Strickland: Inspiration][strickland-inspiration]
25. [Strickland: Two-Stage Sync/Async Validation][strickland-two-stage]

## Phase 5: Integration History and Working Prototype

26. [Chapter 11: The History of DataAnnotations Integration Across .NET](11-integration-history.md) — Chronological history of every integration point with PR/issue citations
27. [Chapter 12: The Async Validation Prototype — A Working Demo](12-async-validation-demo.md) — Analysis of the [`oroztocil/validation-demo`][validation-demo-branch] branch
28. [Microsoft.Extensions.Validation design issue (dotnet/aspnetcore#46349)][aspnetcore-46349] — Unified validation design proposal
29. [Minimal API validation PR (dotnet/aspnetcore#60724)][aspnetcore-60724] — Core implementation of Minimal API validation

<nav>

<a href="README.md">Table of Contents</a> | <a href="01-built-in-validation-attributes.md">Next: Built-In Validation Attributes →</a>

</nav>

<!-- Reference definitions -->

[blog-standard-validators]: https://jeffhandley.com/2010-09-22/riaservicesstandardvalidators
[blog-custom-methods]: https://jeffhandley.com/2010-09-26/riaservicescustomvalidationmethods
[blog-custom-reusable]: https://jeffhandley.com/2010-09-26/riaservicescustomreusablevalidators
[blog-attribute-propagation]: https://jeffhandley.com/archive/2010/09/30/RiaServicesValidationAttributePropagation.aspx
[blog-validation-triggers]: https://jeffhandley.com/archive/2010/10/06/RiaServicesValidationTriggers.aspx
[blog-cross-field]: https://jeffhandley.com/2010-10-10/crossfieldvalidation
[blog-entity-level]: https://jeffhandley.com/2010-10-12/entitylevelvalidation
[blog-providing-context]: https://jeffhandley.com/2010-10-25/riaservicesvalidationcontext
[blog-cross-entity]: https://jeffhandley.com/2010-10-25/crossentityvalidation
[blog-viewmodel-validation]: https://jeffhandley.com/archive/2011/09/06/ViewModelValidation.aspx
[github-ria-validation]: https://github.com/jeffhandley/RIAServicesValidation
[blog-video-tutorials]: https://jeffhandley.com/2010-11-20/riaservicesvalidationvideotutorials
[api-namespace]: https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations?view=net-9.0
[api-validator]: https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validator?view=net-9.0
[api-validation-attribute]: https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute?view=net-9.0
[api-validation-context]: https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationcontext?view=net-9.0
[api-validation-result]: https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationresult?view=net-9.0
[docs-mvc-validation]: https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation
[docs-blazor-validation]: https://learn.microsoft.com/en-us/aspnet/core/blazor/forms/validation
[docs-options-validation]: https://learn.microsoft.com/en-us/dotnet/core/extensions/options
[docs-ef-core]: https://learn.microsoft.com/en-us/ef/core/modeling/entity-properties
[docs-openapi]: https://learn.microsoft.com/en-us/aspnet/core/fundamentals/openapi/aspnetcore-openapi
[docs-wpf-validation]: https://learn.microsoft.com/en-us/dotnet/desktop/wpf/data/
[docs-winforms-validation]: https://learn.microsoft.com/en-us/dotnet/desktop/winforms/input-keyboard/validation
[strickland-docs]: https://strickland.io
[strickland-github]: https://github.com/jeffhandley/strickland
[strickland-inspiration]: https://github.com/jeffhandley/strickland/blob/master/docs/inspiration.md
[strickland-two-stage]: https://github.com/jeffhandley/strickland/blob/master/docs/async-validation/two-stage-sync-async-validation.md
[validation-demo-branch]: https://github.com/dotnet/aspnetcore/tree/oroztocil/validation-demo
[aspnetcore-46349]: https://github.com/dotnet/aspnetcore/issues/46349
[aspnetcore-60724]: https://github.com/dotnet/aspnetcore/pull/60724
