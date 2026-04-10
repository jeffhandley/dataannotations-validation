# Chapter 13: Object Graph Validation — Recursive Walks and Result Representation

<nav>

<a href="12-async-validation-demo.md">← Previous: The Async Validation Prototype</a> | <a href="README.md">Table of Contents</a> | <a href="appendix-a-integration-points.md">Next: Appendix A: .NET Integration Points Catalog →</a>

</nav>

> **Key Concept:** The core `Validator` class does **not** recurse into nested objects. Each invocation model handles object graph validation differently — from "not at all" to full recursive walks with cycle detection and depth limits.

## Why This Matters for Async Validation

When adding async validation to DataAnnotations, the design must account for object graphs. If a `Person` has an `Address` property, and `Address` has a `[ZipCodeValid]` async validator, then any invocation model that walks into `Address` must also support async validation at that depth. Conversely, invocation models that only validate the top-level object may not need graph-aware async support — but they also can't validate the full graph.

Understanding who recurses, how deep, and how results are keyed is essential context for the async validation project plan.

## The Core: `Validator.TryValidateObject()` — Single Level Only

The foundation of all DataAnnotations validation, `Validator.TryValidateObject()`, is explicitly documented as **non-recursive**:

> *"This method evaluates all ValidationAttributes attached to the object instance's type... This process is not recursive."*
> — [Validator.cs source comments][validator-cs]

### What `validateAllProperties` Actually Controls

The `validateAllProperties` parameter is commonly misunderstood as controlling recursion depth. It does not:

| `validateAllProperties` | Behavior |
|-------------------------|----------|
| `false` (default) | Only `[Required]` attributes on immediate properties are checked |
| `true` | All `ValidationAttribute`s on immediate properties are checked |

In both cases, validation **never** descends into nested objects or collections.

### The Three-Step Pipeline (Per Object)

```
Step 1: Validate immediate property-level attributes
          ↓ (only if Step 1 passes)
Step 2: Validate type-level (class) attributes
          ↓ (only if Step 2 passes)
Step 3: Call IValidatableObject.Validate() if implemented
```

Each step short-circuits — if property validation fails, type-level and `IValidatableObject` are skipped entirely.

### What This Means

```csharp
public class Order
{
    [Required]
    public string OrderNumber { get; set; }

    [Required]
    public Address ShippingAddress { get; set; } // [Required] checks non-null only

    public List<OrderItem> Items { get; set; }   // Not validated at all
}

public class Address
{
    [Required]
    public string Street { get; set; }  // Never reached by Validator

    [Required]
    [RegularExpression(@"^\d{5}$")]
    public string ZipCode { get; set; } // Never reached by Validator
}
```

When you call `Validator.TryValidateObject(order, context, results, validateAllProperties: true)`:

- ✅ `Order.OrderNumber` — validated (immediate property)
- ✅ `Order.ShippingAddress` — only checks non-null (the `[Required]` attribute)
- ❌ `Address.Street` — **not validated** (nested object, no recursion)
- ❌ `Address.ZipCode` — **not validated**
- ❌ `Order.Items` elements — **not validated** (collection not iterated)

## Invocation Model Comparison

### ASP.NET Core MVC — Full Recursive Graph Walk

MVC is the **only built-in invocation model** that performs full recursive object graph validation out of the box.

**How it works:** `ValidationVisitor` implements a visitor pattern that walks the entire model tree:

1. `ObjectModelValidator.Validate()` creates a `ValidationVisitor` and starts at the root model
2. For each node, `ModelMetadata` determines the type:
   - **Simple types** — validated directly via `DataAnnotationsModelValidator`
   - **Complex types** (`ModelMetadata.IsComplexType`) — children are visited recursively via `VisitComplexType()`
   - **Enumerables** (`ModelMetadata.IsEnumerableType`) — each element is visited via `DefaultCollectionValidationStrategy`, generating indexed keys like `Items[0]`
3. At each node, all registered `IModelValidator` implementations run (including `DataAnnotationsModelValidator` and `ValidatableObjectAdapter`)

**Cycle detection:** `ValidationVisitor` maintains a `_currentPath` stack. If a model instance is already on the path, traversal stops for that branch.

**Depth control:** `MvcOptions.MaxValidationDepth` (default: 32) caps recursion depth. Exceeding it throws `InvalidOperationException`. This was [added in 2019][aspnetcore-pr-7614] to prevent stack overflows from deeply nested or self-referencing graphs.

**Result representation:** Validation errors include the full property path, built by `ModelNames.CreatePropertyModelName()`:

```
Address.Street           → nested property
Items[0].Name            → collection element property
Items[2].Address.ZipCode → deeply nested through collection
```

**`ValidationStateDictionary`** can override traversal behavior per model instance — suppressing validation, changing the key, or substituting metadata.

### Blazor — Top-Level Only (by Default)

Blazor's `DataAnnotationsValidator` validates **only the root model**:

- **On field change:** `Validator.TryValidateProperty()` on the changed property
- **On form submit:** `Validator.TryValidateObject(editContext.Model, ..., validateAllProperties: true)` — single level, no recursion

**Validation keys are `FieldIdentifier`**, which stores `(Model, FieldName)` — not a path. This means `FieldIdentifier` for `Address.Street` would reference the `Address` object instance, not a path from the root.

**The `ObjectGraphDataAnnotationsValidator` experiment:**

An experimental package (`Microsoft.AspNetCore.Components.DataAnnotations.Validation`) shipped as a preview and provided `ObjectGraphDataAnnotationsValidator` with a `[ValidateComplexType]` attribute. It supported recursive validation of nested objects and `IList<T>` collections but [not dictionaries][aspnetcore-44143]. This package was never promoted to stable.

The current approach in .NET 10 is to use `Microsoft.Extensions.Validation` (see below) which provides graph-aware validation through `ValidatableTypeInfo`.

### Options Validation — Opt-In Recursion via Attributes

`DataAnnotationValidateOptions<T>` calls `Validator.TryValidateObject()` on the root options object — **single level only** by default.

Recursive validation of nested objects requires explicit opt-in:

| Attribute | Purpose |
|-----------|---------|
| `[ValidateObjectMembers]` | Recurse into a nested object and validate its properties |
| `[ValidateEnumeratedItems]` | Iterate a collection and validate each element |

**Source generator behavior:** The Options Validation Source Generator (`OptionsValidatorGenerator`) detects these attributes in `Parser.cs`, synthesizes validators for nested types, and emits validation calls in `Emitter.cs`. It also emits a diagnostic warning when a complex member property appears to need transitive validation but isn't annotated — nudging developers to be explicit.

**Result representation:** `ValidateOptionsResult` contains a flat list of failure strings. Nested paths are represented as `"DataAnnotation validation failed for 'ShippingAddress.Street' members: 'Street' with the error: 'The Street field is required.'"`.

### Microsoft.Extensions.Validation (.NET 10) — Full Recursive Graph Walk

The newest validation framework supports **full recursive object graph validation** by design:

1. `ValidatableTypeInfo` describes a type and its `Members` (as `ValidatablePropertyInfo`)
2. `ValidatablePropertyInfo.ValidateAsync()` inspects each property value at runtime:
   - If the property value's type has a registered `IValidatableInfo`, it recurses
   - If the property value is enumerable, it validates each element
3. `ValidationOptions.MaxDepth` guards against infinite recursion

**Result representation:** Errors are keyed by dot-separated paths built from `ValidateContext.CurrentValidationPath`:

```
Parent.Child              → nested object
Parent.Items[0]           → collection element
Parent.Items[0].Street    → property on collection element
```

Errors are stored in `ValidationErrors` (a dictionary of path → error messages), and `ValidationErrorContext.Path` carries the full path for each error.

**Important:** The graph walk happens at the `ValidatableTypeInfo` level, not in the core `Validator` class. The `Validator` is still called for individual nodes — it does not gain recursion itself.

### EF Core — Metadata Only (No Runtime Validation)

EF Core reads DataAnnotations attributes as **metadata for model configuration**, not for runtime validation. `[Required]` maps to non-nullable columns, `[MaxLength]` maps to column length, etc. EF Core does not call `Validator.TryValidateObject()` and does not perform any object graph validation.

### OpenAPI Schema Generation — Metadata Only

OpenAPI reads validation attributes for schema generation (`minLength`, `maximum`, `pattern`, etc.) but does not invoke runtime validation.

## Ecosystem Projects That Fill the Gap

Several projects exist specifically because the core `Validator` doesn't recurse:

### MiniValidation (Damian Edwards)

[MiniValidation][minivalidation] is a minimalist library built directly atop `System.ComponentModel.DataAnnotations` that adds **recursive graph validation**:

- **Recursion:** Enabled by default (`recurse: true`). Walks nested objects and collections.
- **Cycle detection:** Uses a `Dictionary<object, bool?>` keyed by object identity to detect and skip cycles.
- **Max depth:** `MiniValidator.MaxDepth` (default: 32) — configurable.
- **Collections:** `IEnumerable` items are iterated and validated individually.
- **Skip recursion:** `[SkipRecursion]` attribute opts a property out of recursive validation.
- **Async support:** `TryValidateAsync()` supports `IAsyncValidatableObject` (MiniValidation's own interface).
- **Result representation:** `IDictionary<string, string[]>` with dot-separated path keys:

```csharp
var (isValid, errors) = MiniValidator.TryValidate(order);
// errors["ShippingAddress.Street"] = ["The Street field is required."]
// errors["Items[0].Name"] = ["The Name field is required."]
```

MiniValidation was created by [Damian Edwards][damian-edwards], a principal architect on ASP.NET Core at Microsoft. It demonstrates the pattern that any built-in recursive validation should follow.

### RecursiveDataAnnotationsValidation

[RecursiveDataAnnotationsValidation][recursive-daval] provides `RecursiveDataAnnotationValidator.TryValidateObjectRecursive()` as a drop-in recursive wrapper around `Validator.TryValidateObject()`:

- Recurses through sub-objects and `IEnumerable` collections
- `[SkipRecursiveValidation]` attribute opts out per property
- Prefixes member names for nested error paths

### DataAnnotationsValidatorRecursive

[DataAnnotationsValidatorRecursive][daval-recursive] offers a similar `TryValidateObjectRecursive<T>()`:

- Walks `IEnumerable` collections and nested objects
- Prefixes member names as `Parent.Child`
- Simpler implementation without cycle detection

### FluentValidation

[FluentValidation][fluentvalidation] takes a fundamentally different approach — validators are separate classes rather than attributes — but it solves the graph problem elegantly:

- **Nested objects:** `RuleFor(x => x.Address).SetValidator(new AddressValidator())`
- **Collections:** `RuleForEach(x => x.Items).SetValidator(new OrderItemValidator())`
- **Child rules:** `RuleFor(x => x.Address).ChildRules(address => { ... })`
- **Result paths:** Include property chains and collection indexes (`Orders[0].Total`)

FluentValidation also offers `FluentValidation.AspNetCore` for MVC integration and has its own `IAsyncValidator` interface.

## Summary: Graph Walk Behavior by Invocation Model

| Invocation Model | Recurses? | Collections? | Cycle Detection? | Max Depth | Result Path Format |
|------------------|-----------|--------------|-------------------|-----------|--------------------|
| **`Validator.TryValidateObject()`** | ❌ No | ❌ No | N/A | N/A | `MemberName` (single level) |
| **ASP.NET Core MVC** | ✅ Yes | ✅ Yes (indexed) | ✅ Yes (path stack) | 32 (configurable) | `Address.Street`, `Items[0].Name` |
| **Blazor (default)** | ❌ No | ❌ No | N/A | N/A | `FieldIdentifier(model, name)` |
| **Blazor (experimental)** | ✅ Yes | ✅ `IList<T>` only | Unknown | Unknown | `FieldIdentifier` per nested object |
| **Options.DataAnnotations** | ⚠️ Opt-in | ⚠️ Opt-in | N/A | N/A | Flat failure strings |
| **Options Source Generator** | ⚠️ Opt-in | ⚠️ Opt-in | Warns if missing | N/A | Flat failure strings |
| **M.E.Validation (.NET 10)** | ✅ Yes | ✅ Yes (indexed) | ✅ Yes (MaxDepth) | Configurable | `Parent.Child`, `Items[0].Name` |
| **EF Core** | N/A (metadata) | N/A | N/A | N/A | N/A |
| **OpenAPI** | N/A (metadata) | N/A | N/A | N/A | N/A |
| **MiniValidation** | ✅ Yes | ✅ Yes (indexed) | ✅ Yes (identity dict) | 32 (configurable) | `Address.Street`, `Items[0].Name` |

## Implications for Async Validation Design

1. **The core `Validator` class needs a recursion story.** Adding `TryValidateObjectAsync()` without recursion means async validators on nested objects are unreachable through the core API. Consider whether the new async methods should support an opt-in recursion parameter.

2. **MVC already recurses — its async path must too.** Since `ValidationVisitor` walks the full graph, any async version must also walk the full graph asynchronously. This means `DataAnnotationsModelValidator` must support async, and the visitor pattern must handle `await` at each node.

3. **Blazor's gap is already being addressed.** The move to `Microsoft.Extensions.Validation` in .NET 10 gives Blazor graph-aware validation. The async path should flow naturally from that framework.

4. **Options validation needs graph-aware async.** The `[ValidateObjectMembers]` and `[ValidateEnumeratedItems]` attributes already define the recursion shape. The source generator must emit async validation calls for nested types that have async validators.

5. **Result representation must be consistent.** Any async validation that recurses should use the same path-based error keys that MVC and `Microsoft.Extensions.Validation` use: `Address.Street`, `Items[0].Name`, etc.

6. **Cycle detection is mandatory for async.** Async validators could take arbitrary time. Without cycle detection, a recursive graph with async validators could produce infinite or very long-running validation. Every recursive async path must have both cycle detection and depth limits.

7. **Ecosystem libraries will need updating.** MiniValidation, FluentValidation, and other graph-aware libraries will need to integrate with the new async DataAnnotations validators to maintain their value.

<nav>

<a href="12-async-validation-demo.md">← Previous: The Async Validation Prototype</a> | <a href="README.md">Table of Contents</a> | <a href="appendix-a-integration-points.md">Next: Appendix A: .NET Integration Points Catalog →</a>

</nav>

<!-- Reference definitions -->

[validator-cs]: https://github.com/dotnet/runtime/blob/main/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations/Validator.cs
[aspnetcore-pr-7614]: https://github.com/dotnet/aspnetcore/pull/7614
[aspnetcore-44143]: https://github.com/dotnet/aspnetcore/issues/44143
[minivalidation]: https://github.com/DamianEdwards/MiniValidation
[damian-edwards]: https://github.com/DamianEdwards
[recursive-daval]: https://github.com/tgharold/RecursiveDataAnnotationsValidation
[daval-recursive]: https://github.com/reustmd/DataAnnotationsValidatorRecursive
[fluentvalidation]: https://github.com/FluentValidation/FluentValidation
