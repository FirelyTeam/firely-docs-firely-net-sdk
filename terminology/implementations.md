# Terminology service implementations

## The operations

`ITerminologyService` defines a method per FHIR terminology operation. Each takes a `Parameters` resource — built with the matching helper class — and returns the operation's output as a `Parameters` or a single `Resource`. Most operations also accept an optional `id` and a `useGet` flag (which makes an external client use `GET` instead of `POST`).

| Method | FHIR operation | Input helper |
|--------|----------------|--------------|
| `ValueSetValidateCode` | `$validate-code` on ValueSet | `ValidateCodeParameters` |
| `CodeSystemValidateCode` | `$validate-code` on CodeSystem | `ValidateCodeParameters` |
| `Expand` | `$expand` | `ExpandParameters` |
| `Lookup` | `$lookup` | `LookupParameters` |
| `Translate` | `$translate` | `TranslateParameters` |
| `Subsumes` | `$subsumes` | `SubsumesParameters` |
| `Closure` | `$closure` | `ClosureParameters` |

## LocalTerminologyService

`LocalTerminologyService` handles terminology in-process, without calling a third party — it does as much as it can itself.

```{note}
The `LocalTerminologyService` does not support every terminology feature; fully supporting them requires a lot of terminology expertise. For advanced features, use an `ExternalTerminologyService` and let a dedicated terminology server handle the request.
```

It requires an `IResourceResolver` to find the FHIR resources it needs. The following operation is currently supported:

- `ValueSetValidateCode` — validates that a coded value is in the set of codes allowed by a value set. The value set is found through the provided `IResourceResolver`, for example a {ref}`FhirPackageSource<fhir-package-source>` containing all the artifacts from a package.

In some cases a `ValueSet` is defined implicitly by the `valueSet` element of a `CodeSystem` (an implicit value set of all the codes in that code system). If the `IResourceResolver` also implements `IConformanceSource`, `LocalTerminologyService` can validate against such an implicitly defined value set too.

## ExternalTerminologyService

`ExternalTerminologyService` implements all of `ITerminologyService` and uses a `FhirClient` to delegate every operation to an external terminology server.

Each operation accepts a `bool useGet = false` parameter; set it to `true` to make the client use the `GET` REST operation instead of `POST`.

See {doc}`basic-usage` for an example that expands a value set through an external server.

## CustomValueSetTerminologyService

`CustomValueSetTerminologyService` is an abstract `ITerminologyService` that validates codes against a fixed, code-defined value set. The base class implements most of the machinery; to define your own you implement `ValidateCodeType` and supply a few values through the constructor:

- `ValidateCodeType` — validates a single string against the custom value set, returning `true` when the code is valid.
- `terminologyType` — a human-readable name of the code type, used only in error messages.
- `codeSystem` — the name of the specification defining the value set's members.
- `codeValueSets` — the canonical URLs of the value set (more than one if a FHIR version changed it).

Two implementations ship with the SDK:

- `MimeTypeTerminologyService` — verifies that a code is a valid MIME type.
- `LanguageTerminologyService` — verifies that a code is a valid language code.

`LanguageTerminologyService` is a compact example:

```csharp
public class LanguageTerminologyService : CustomValueSetTerminologyService
{
    private const string LANGUAGE_SYSTEM = "urn:ietf:bcp:47";
    public const string LANGUAGE_VALUESET = "http://hl7.org/fhir/ValueSet/all-languages";

    public LanguageTerminologyService() : base("language", LANGUAGE_SYSTEM, [LANGUAGE_VALUESET])
    {
    }

    override protected bool ValidateCodeType(string code)
    {
        var regex = new Regex("^[a-z]{2}(-[A-Z]{2})?$"); // two lowercase letters, optionally a dash and two uppercase letters
        return regex.IsMatch(code);
    }
}
```
