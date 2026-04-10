# System.ComponentModel.DataAnnotations Validation Training

This repository contains comprehensive training material for .NET's `System.ComponentModel.DataAnnotations` validation system. It covers everything from built-in attributes through advanced extensibility, and explores the path toward adding async validation to DataAnnotations. The material is designed to bring a team up to speed so they can build a project plan for adding async validation across the entire .NET product suite.

## Table of Contents

1. **[Suggested Reading Order](00-suggested-reading-order.md)** — External resources organized by learning phase
2. **[Chapter 1: Built-In Validation Attributes](01-built-in-validation-attributes.md)** — Complete catalog of attributes shipped with .NET
3. **[Chapter 2: Creating Custom Validation Attributes](02-creating-custom-attributes.md)** — Deriving from `ValidationAttribute` for reusable rules
4. **[Chapter 3: Annotating Objects for Validation](03-annotating-objects.md)** — Applying attributes to properties and classes
5. **[Chapter 4: Programmatic (Manual) Validation](04-programmatic-validation.md)** — Using the `Validator` class directly
6. **[Chapter 5: ASP.NET MVC Automatic Validation](05-aspnet-mvc-validation.md)** — How model binding triggers validation
7. **[Chapter 6: Advanced Custom Validation](06-advanced-custom-validation.md)** — `IValidatableObject` and complex validation scenarios
8. **[Chapter 7: ValidationContext Deep Dive](07-validation-context.md)** — Services, Items dictionary, and context propagation
9. **[Chapter 8: The ValidationResult API](08-validation-result-api.md)** — Understanding validation results and error reporting
10. **[Chapter 9: The Async Validation Gap](09-async-validation-gap.md)** — Why DataAnnotations lacks async support today
11. **[Chapter 10: Strickland — Parallel Concepts and Async Validation](10-strickland.md)** — Lessons from a JavaScript validation framework
12. **[Chapter 11: The History of DataAnnotations Integration Across .NET](11-integration-history.md)** — Chronological history of every integration point
13. **[Chapter 12: The Async Validation Prototype — A Working Demo](12-async-validation-demo.md)** — Analysis of a working async validation prototype
14. **[Appendix A: .NET Integration Points Catalog](appendix-a-integration-points.md)** — Every framework that consumes DataAnnotations
15. **[Appendix B: Complete Reference Link Library](appendix-b-references.md)** — All source links and documentation references

## About This Material

- Content is drawn from source code analysis of [dotnet/runtime][dotnet-runtime], official Microsoft documentation, the [RIA Services Validation blog series][ria-series] (2010), and the [Strickland][strickland] JavaScript validation framework
- The material targets team members who will work on adding async validation to DataAnnotations

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

[dotnet-runtime]: https://github.com/dotnet/runtime
[ria-series]: https://jeffhandley.com/2010-09-22/riaservicesstandardvalidators
[strickland]: https://github.com/jeffhandley/strickland
