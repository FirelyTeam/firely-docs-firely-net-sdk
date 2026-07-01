(profile-validation-internals)=
# How profile validation works

The `Validator` described in {doc}`profile-validation` is a convenience wrapper over a lower-level engine: it compiles `StructureDefinition`s into schemas, runs them against an instance, and converts the result to an `OperationOutcome`. This page explains that engine and — more usefully — how to *tune* it, by configuring its pieces and handing them to the `Validator`.

```{note}
The engine's types — the schema resolver, `ISchemaBuilder`, and the assertion types — are currently marked experimental in the SDK (`[Experimental]` on .NET 8+, `[Obsolete]` on older targets), so you may need to suppress that diagnostic to use them; the designation is under review. `ValidationSettings` and the `Validator` itself are not experimental.
```

## The engine

A `StructureDefinition` is compiled into an `ElementSchema` — a tree of `IAssertion`s, one per constraint (cardinality, binding, type, FhirPath invariant, slicing, and so on). Validation runs a schema against an instance to produce a `ResultReport`, which the `Validator` converts to a FHIR `OperationOutcome`. The `Validator` supplies the three things the engine needs: a schema resolver (which compiles and caches StructureDefinitions), a terminology service, and a `ValidationSettings`.

The engine lives in two assemblies: `Firely.Fhir.Validation` (the assertions, `ElementSchema`, and `ResultReport`) and `Firely.Fhir.Validation.Compilation` (compiling StructureDefinitions into schemas).

## Schema compilation and caching

`StructureDefinitionToElementSchemaResolver` compiles StructureDefinitions — fetched from an `IAsyncResourceResolver` — into `ElementSchema`s (it implements `IElementSchemaResolver`). Compilation is expensive, so how you create the resolver matters:

- `CreatedCached(source)` — caches compiled schemas; this is what the `Validator` builds by default.
- `CreatedCached(source, cache)` — caches into a `ConcurrentDictionary` you supply, so the cache can be shared across validators.
- `Create(source)` — no caching.

You can also extend compilation with your own rules by passing extra `ISchemaBuilder`s, whose `Build(...)` method contributes additional `IAssertion`s to the compiled schema.

## Tuning the validator

You reach these knobs not by calling the engine yourself, but by passing a custom schema resolver and/or `ValidationSettings` to the `Validator`'s constructor:

```csharp
public Validator(
    IAsyncResourceResolver resourceResolver,
    ICodeValidationTerminologyService terminologyService,
    IExternalReferenceResolver? referenceResolver = null,
    ValidationSettings? settings = null,
    IElementSchemaResolver? schemaResolver = null)   // inject a tuned resolver
```

For example, to supply a resolver (whose caching and custom `ISchemaBuilder`s you control) and treat best-practice constraints as errors:

```csharp
var resolver = StructureDefinitionToElementSchemaResolver.CreatedCached(source);
var settings = new ValidationSettings { ConstraintBestPractices = ValidateBestPracticesSeverity.Error };

var validator = new Validator(source, terminologyService, settings: settings, schemaResolver: resolver);
var outcome = validator.Validate(patient);
```

You still call the `Validator`'s POCO-based `Validate` (see {doc}`profile-validation`); the customization lives in the resolver and settings you inject.

```{note}
There is currently no convenient public entry point to run a compiled schema directly against a POCO — the assertion-level `Validate` methods either take an `ITypedElement` or require an internal `ValidationState`. So the lower-level tuning is done through the `Validator` as shown above rather than by invoking the engine yourself; a convenient POCO entry point is tracked in [firely-validator-api#669](https://github.com/FirelyTeam/firely-validator-api/issues/669).
```

## Why go lower-level

- **Caching** — share a compiled-schema cache across validators, or disable caching, instead of accepting the default.
- **Custom compilation** — add your own `ISchemaBuilder`s to contribute extra assertions to compiled schemas.
- **Validation settings** — configure the `ValidationSettings` the `Validator` would otherwise build with defaults (see {doc}`profile-validation`).
