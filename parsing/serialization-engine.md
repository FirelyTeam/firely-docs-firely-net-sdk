(serialization-engine)=
# The serialization engine

`IFhirSerializationEngine` is a small facade that bundles JSON and XML, serialization and deserialization, behind a single object. It is what `FhirClient` uses internally, and it is handy when you just want to move whole resources to and from strings without picking a specific (de)serializer.

```csharp
var engine = FhirSerializationEngineFactory.Recoverable(ModelInfo.ModelInspector);

Resource? patient = engine.DeserializeFromJson(json);
string xml = engine.SerializeToXml(patient);
```

The interface has exactly four methods:

| Method | Purpose |
|--------|---------|
| `DeserializeFromJson(string)` | JSON string → resource |
| `DeserializeFromXml(string)` | XML string → resource |
| `SerializeToJson(Resource)` | resource → JSON string |
| `SerializeToXml(Resource)` | resource → XML string |

```{note}
The engine is deliberately minimal: it works only with **strings** and only with **resources**. If you need to read from or write to a reader/writer, handle datatypes, or collect issues without an exception, use the {doc}`deserialization` and {doc}`serialization` classes directly — the engine is a convenience layer over exactly those classes.
```

## Creating an engine

`FhirSerializationEngineFactory` creates an engine for a given `ModelInspector`, preconfigured for one of the deserialization modes (the same modes described in {doc}`error-handling`):

- `Strict` — report every issue
- `Recoverable` — ignore issues that do not lose data
- `BackwardsCompatible` — also accept unknown elements/codes from other FHIR versions
- `SyntaxOnly` — only check the JSON/XML syntax
- `Ostrich` — ignore everything
- `Custom(…)` — supply your own JSON and XML settings

## Legacy engines

`FhirSerializationEngineFactory.Legacy` offers engines backed by the older, `ElementModel`-based (de)serializers (`Strict`, `Permissive`, `BackwardsCompatible`, `Ostrich`). These exist for backward compatibility; see {doc}`ElementModel <elementmodel>`.
