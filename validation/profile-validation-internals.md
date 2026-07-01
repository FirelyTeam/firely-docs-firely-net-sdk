(profile-validation-internals)=
# How profile validation works

The `Validator` described in {doc}`profile-validation` is a convenience wrapper over a lower-level engine: it compiles `StructureDefinition`s into schemas, runs them against an instance, and converts the result to an `OperationOutcome`. This page explains that engine and how to drive it directly — useful when you need control the wrapper does not expose, such as tuning schema caching, adding custom compilation rules, or working with the raw validation result.

```{note}
This lower-level API is currently marked experimental in the SDK — depending on your target framework you may need to suppress an `ExperimentalApi` (or obsolete) diagnostic to call it. It has been stable in practice, and the designation is being reviewed.
```

## The engine

A `StructureDefinition` is compiled into an `ElementSchema` — a tree of `IAssertion`s, one per constraint (cardinality, binding, type, FhirPath invariant, slicing, and so on). Validation runs a schema against an instance and produces a `ResultReport`. The `Validator` wraps three things you otherwise supply yourself: a schema resolver (which compiles and caches StructureDefinitions), a terminology service, and a `ValidationSettings`.

The engine lives in two assemblies: `Firely.Fhir.Validation` (the assertions, `ElementSchema`, and `ResultReport`) and `Firely.Fhir.Validation.Compilation` (compiling StructureDefinitions into schemas).

## Compiling schemas

`StructureDefinitionToElementSchemaResolver` compiles StructureDefinitions — fetched from an `IAsyncResourceResolver` — into `ElementSchema`s. It implements `IElementSchemaResolver`, whose `GetSchema(Canonical)` returns the compiled schema for a canonical URL.

Compilation is expensive, so how you create the resolver matters:

- `CreatedCached(source)` — caches compiled schemas; this is what the `Validator` uses internally.
- `CreatedCached(source, cache)` — same, but you supply the `ConcurrentDictionary` cache, so it can be shared across resolvers.
- `Create(source)` — no caching.

You can also extend compilation with your own rules by passing extra `ISchemaBuilder`s, whose `Build(...)` method contributes additional `IAssertion`s to the schema.

## Running validation directly

```csharp
var resolver = StructureDefinitionToElementSchemaResolver.CreatedCached(source);
var settings = new ValidationSettings(resolver, terminologyService);

var schema = resolver.GetSchema(Canonical.ForCoreType("Patient"));
var node = patient.ToPocoNode(ModelInfo.ModelInspector);
var report = ((IValidatable)schema).Validate(node, settings, new ValidationState());
```

A schema is an `IAssertion`, and the `IValidatable` / `IGroupValidatable` interfaces it implements expose the `Validate` methods — one taking a single `PocoNode`, one taking a set. You pass the instance as a `PocoNode` (via `ToPocoNode`), the `ValidationSettings` — the same type described in {doc}`profile-validation`, constructed directly here with its public constructor — and a fresh `ValidationState` (the per-run state the engine threads through validation). The call returns a `ResultReport`.

## The ResultReport

`ResultReport` is the raw result the engine produces — the `Validator` simply calls `ToOperationOutcome()` on it. Its members:

- `Result` — `Success`, `Failure`, or `Undecided`; `IsSuccessful` is the shorthand for success.
- `Errors` / `Warnings` — the `IssueAssertion`s of that severity (`GetIssues(severity)` for other severities).
- `Evidence` — all the assertions gathered as evidence for the result.
- `Combine(reports)` — merges several reports (for example, from validating against multiple profiles) into one, taking the weakest result and the combined evidence.
- `ToOperationOutcome()` — converts the report to a FHIR `OperationOutcome`.

## Why use the lower-level API

- **Caching** — share a schema cache across validators, or disable caching, rather than accepting the wrapper's fixed behaviour.
- **Custom compilation** — add your own `ISchemaBuilder`s to contribute extra assertions to compiled schemas.
- **Working with schemas** — obtain, inspect, or reuse compiled `ElementSchema`s directly.
- **Result handling** — combine or filter `ResultReport`s before (or instead of) converting to an `OperationOutcome`.
