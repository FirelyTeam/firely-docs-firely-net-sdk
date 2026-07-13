# The IAsyncResourceResolver interface

The `IAsyncResourceResolver` interface is used to resolve conformance resources from the specification or an implementation guide
(e.g. StructureDefinition, ValueSet, CodeSystem):

```csharp
public interface IAsyncResourceResolver
{
    /// <summary>Find a resource based on its relative or absolute uri.</summary>
    Task<ResolverResult> TryResolveByUriAsync(string uri);

    /// <summary>Find a (conformance) resource based on its canonical uri.</summary>
    Task<ResolverResult> TryResolveByCanonicalUriAsync(string uri);
}
```

The principal method is `TryResolveByCanonicalUriAsync`, which resolves a conformance resource by its canonical url. `TryResolveByUriAsync` is
used by some implementations to resolve non-conformance resources, like example instances of resources, but is otherwise not widely supported and
is normally just hardwired to use the canonical lookup.

```{note}
Earlier versions of the interface exposed `ResolveByUriAsync` and `ResolveByCanonicalUriAsync`, which returned the resource or `null`. These are now
**obsolete**: a `null` result did not distinguish "not found" from other kinds of failure. Prefer the `TryResolve…` methods, which return a
`ResolverResult`.
```

## ResolverResult

`ResolverResult` reports either the resolved resource or the reason resolution failed:

- `Value` — the resolved `Resource`, or `null` on failure.
- `Success` — `true` when a resource was found (`ResolverResult` also converts implicitly to `bool`).
- `Error` — a `ResolverException` describing the failure (e.g. not found) when `Success` is `false`.

```csharp
var result = await resolver.TryResolveByCanonicalUriAsync("http://hl7.org/fhir/StructureDefinition/Patient");
if (result.Success)
{
    var sd = (StructureDefinition)result.Value!;
}
else
{
    // inspect result.Error
}
```

## Convenience extension methods

There are a number of useful extension methods that simplify the use of the `IAsyncResourceResolver` interface:

* `FindStructureDefinitionAsync(this IAsyncResourceResolver resolver, string uri)` — resolves a StructureDefinition.
* `FindStructureDefinitionForCoreTypeAsync(this IAsyncResourceResolver resolver, string coreType)` — resolves a StructureDefinition for a named FHIR resource or datatype.
* `FindValueSetAsync(this IAsyncResourceResolver resolver, string uri)` — resolves a ValueSet.
* `FindCodeSystemAsync(this IAsyncResourceResolver resolver, string uri)` — resolves a CodeSystem.

These methods return the resolved resource, or `null` if the resource is not found or is not of the requested type.

## IConformanceSource

There is a related interface `IConformanceSource`, which extends `ICommonConformanceSource`. It has methods to find specific types of conformance
resources using criteria other than the canonical uri:

```csharp
public interface IConformanceSource : ICommonConformanceSource
{
    /// <summary>Find a CodeSystem resource by a ValueSet canonical url that contains all codes from that codesystem.</summary>
    CodeSystem FindCodeSystemByValueSet(string valueSetUri);

    /// <summary>List all resource uris for the resources managed by the source, optionally filtered by type. (these are not Canonical Uris)</summary>
    IEnumerable<string> ListResourceUris(ResourceType? filter = null);

    /// <summary>Find ConceptMap resources which map from the given source to the given target.</summary>
    IEnumerable<ConceptMap> FindConceptMaps(string? sourceUri = null, string? targetUri = null);

    /// <summary>Finds a NamingSystem resource by matching any of a system's UniqueIds.</summary>
    NamingSystem? FindNamingSystem(string uniqueId);
}
```

This interface is not supported by all implementations of `IAsyncResourceResolver`.
