# Composing terminology services

Real applications often combine terminology services: consult a fast, local service first and fall back to an external server, or route specific value sets to a service that specializes in them. `MultiTerminologyService` composes several `ITerminologyService`s for exactly this — and because it is itself an `ITerminologyService`, the combination is used just like any single service.

## Fallback

`MultiTerminologyService` tries its services in the order given. The first service that comes back with a result — success or failure — wins; if a service is *indecisive* (it throws a `FhirOperationException`), the next service is consulted.

```csharp
var local = new LocalTerminologyService(ZipSource.CreateValidationSource());

var client = new FhirClient("https://someterminologyserver.org/fhir");
var external = new ExternalTerminologyService(client);

var multi = new MultiTerminologyService(local, external);
```

Here the local service is always consulted first; only when it cannot decide does the request go to the external server.

## Routing

Sometimes you already know which service should handle certain value sets — for example, a local service that owns all of your custom value sets. Routing sends matching value sets to a chosen service first. Provide a `TerminologyServiceRoutingSettings` per service:

```csharp
var local = new LocalTerminologyService(ZipSource.CreateValidationSource());
var localRouting = new TerminologyServiceRoutingSettings(local)
{
    PreferredValueSets = new string[]{"http://fire.ly/ValueSet/*"}
};

var client = new FhirClient("https://someterminologyserver.org/fhir");
var external = new ExternalTerminologyService(client);
var externalRouting = new TerminologyServiceRoutingSettings(external)
{
    PreferredValueSets = new string[]{"http://hl7.fhir.org/ValueSet/*"}
};

var multi = new MultiTerminologyService(localRouting, externalRouting);
```

```{note}
You can use `*` as a wildcard in the routing patterns.
```

Value sets under `http://fire.ly/ValueSet/` now go to the local service first and those under `http://hl7.fhir.org/ValueSet/` to the external one; anything else follows the order the services were passed to the constructor — here local, then external.
