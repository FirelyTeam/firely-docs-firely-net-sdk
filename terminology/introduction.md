# Using Terminology Services

FHIR defines a set of *terminology operations* — validating a code against a value set, expanding a value set, looking up a concept, translating between value sets, and more. They let an application work with codes and value sets without having to master the details of code systems, value sets, concept maps, and the terminological rules behind them.

The SDK exposes these operations through a single abstraction, `ITerminologyService`, and ships several implementations — from a local, in-process service to one that delegates to an external terminology server.

## The `ITerminologyService` abstraction

Every terminology operation is a method on `ITerminologyService`. Each takes a `Parameters` resource describing the operation's input and returns the operation's output — either another `Parameters` resource or a single `Resource`. A helper class per operation makes building the input easy (for example `ValidateCodeParameters` and `ExpandParameters`).

`ITerminologyService` is itself composed of smaller, role-specific interfaces, so a component can depend on just the slice it needs. The profile validator, for instance, only requires `ICodeValidationTerminologyService` — the validate-code operations.

The operations are:

| Operation | Method | FHIR operation |
|-----------|--------|----------------|
| Validate a code against a value set | `ValueSetValidateCode` | `$validate-code` (ValueSet) |
| Validate a code in a code system | `CodeSystemValidateCode` | `$validate-code` (CodeSystem) |
| Expand a value set | `Expand` | `$expand` |
| Look up a concept | `Lookup` | `$lookup` |
| Translate a code | `Translate` | `$translate` |
| Test subsumption | `Subsumes` | `$subsumes` |
| Maintain a closure table | `Closure` | `$closure` |

Their full signatures are listed under {doc}`/terminology/implementations`.

## Two ways to use them

- **Standalone** — call the operations directly, for example to validate or expand codes in your own code. See {doc}`/terminology/basic-usage`.
- **In validation** — hand a terminology service to the profile validator, which uses it to validate coded values against their bindings. See {ref}`profile-validation`.

## The implementations

- **`LocalTerminologyService`** — resolves and evaluates terminology in-process, using an `IResourceResolver` to find the code systems and value sets. No external dependency, but it supports only a subset of the operations.
- **`ExternalTerminologyService`** — delegates every operation to an external terminology server through a `FhirClient`.
- **`CustomValueSetTerminologyService`** — validates codes against a fixed, code-defined value set (the base for the built-in `MimeTypeTerminologyService` and `LanguageTerminologyService`).
- **`MultiTerminologyService`** — combines several services, with fallback and routing.

See {doc}`/terminology/implementations` for each, and {doc}`/terminology/composing-services` for combining them.
