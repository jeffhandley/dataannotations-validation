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
12. **[Appendix A: .NET Integration Points Catalog](appendix-a-integration-points.md)** — Every framework that consumes DataAnnotations
13. **[Appendix B: Complete Reference Link Library](appendix-b-references.md)** — All source links and documentation references

## About This Material

- Content is drawn from source code analysis of [dotnet/runtime](https://github.com/dotnet/runtime), official Microsoft documentation, Jeff Handley's 2010 RIA Services Validation blog series, and the [Strickland](https://github.com/jeffhandley/strickland) JavaScript validation framework
- DataAnnotations validation saw its most significant innovation during the .NET RIA Services era (2009–2012), when `IValidatableObject`, `ValidationResult`, `ValidationContext`, and `Validator` were designed and refined; Jeff Handley's blog series from that period remains one of the most thorough walkthroughs of the system
- The material targets team members who will work on adding async validation to DataAnnotations

## License

This repository is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
