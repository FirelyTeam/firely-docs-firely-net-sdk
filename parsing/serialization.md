(serialization)=
# Serialization

Serialization turns a POCO back into FHIR JSON or XML. The types below live in:

```csharp
using Hl7.Fhir.Serialization;
```

```{note}
Serialization does **not** validate the POCO — it faithfully writes whatever the object contains, even if that produces invalid FHIR. This is intentional (it allows round-tripping imperfect data and avoids duplicate work), but it means that if you need guaranteed-valid output you should {ref}`validate <validation>` the instance first.
```

## Choosing a serializer

For a single FHIR version, use the ready-made `FhirJsonSerializer.Default` or `FhirXmlSerializer.Default`:

```csharp
var json = FhirJsonSerializer.Default.SerializeToString(patient);
```

For a custom or runtime-chosen model, construct a `BaseFhirJsonSerializer` / `BaseFhirXmlSerializer` with a `ModelInspector` (see {doc}`the ModelInspector </model/model-metadata>`):

```csharp
var serializer = new BaseFhirJsonSerializer(inspector);
```

## Output formats

Serializers produce a string, a UTF-8 byte array, or a document object, and can also write straight to a writer:

```csharp
string json   = FhirJsonSerializer.Default.SerializeToString(patient);
byte[] utf8    = FhirJsonSerializer.Default.SerializeToBytes(patient);
JObject doc    = FhirJsonSerializer.Default.SerializeToDocument(patient);
FhirJsonSerializer.Default.Serialize(patient, utf8JsonWriter);
```

Pass `pretty: true` for indented output (compact is the default):

```csharp
var json = FhirJsonSerializer.Default.SerializeToString(patient, pretty: true);
```

## Summaries

FHIR defines [summary forms](http://hl7.org/fhir/search.html#summary) of a resource — reduced views that contain only certain elements. The simplest way to produce one is to pass a `SummaryType` to `SerializeToString`, optionally restricting to specific top-level elements:

```csharp
var summary = FhirJsonSerializer.Default.SerializeToString(patient, SummaryType.Text);
```

Under the hood a summary is a `SerializationFilter`. You can also create one directly and pass it as a factory to the serializer; the standard filters are available as factory methods:

- `SerializationFilter.ForSummary()` — `_summary=true`
- `SerializationFilter.ForText()` — `_summary=text`
- `SerializationFilter.ForData()` — `_summary=data`
- `SerializationFilter.ForCount()` — `_summary=count`
- `SerializationFilter.ForElements(elements)` — `_elements=…`

The other summary forms mentioned in the FHIR specification need no special serializer support and can be constructed by hand.

```csharp
var json = FhirJsonSerializer.Default.SerializeToString(
    patient, filterFactory: () => SerializationFilter.ForSummary());
```

The filters are highly configurable, and you can write your own by subclassing `SerializationFilter`. For inspiration, see the (fairly simple) built-in implementations such as `ElementMetadataFilter.cs` and `BundleFilter.cs` in the SDK source.

## Convenience extensions

For one-off calls you can skip creating a serializer and use the extension methods on any POCO, which use the default serializer for your FHIR version:

```csharp
string json = patient.ToJson(pretty: true);
string xml  = patient.ToXml();
```

`ToJsonBytes`, `ToXmlBytes` and `ToXDocument` are available too.

## XML

Everything above applies to XML through `FhirXmlSerializer` / `BaseFhirXmlSerializer`; `SerializeToDocument` returns an `XDocument`, and `Serialize` takes an `XmlWriter`:

```csharp
var xml = FhirXmlSerializer.Default.SerializeToString(patient, pretty: true);
```

When you serialize a *datatype* rather than a resource, the XML serializer wraps it in an element named after the type — for example a `HumanName` becomes `<HumanName>…</HumanName>`.

```{admonition} Experimental
:class: warning
As with deserialization, serializing a standalone datatype is outside the FHIR specification (which only defines serialization for resources) and should be treated as experimental. In particular, a lone FHIR primitive has no faithful representation — see {doc}`deserialization`.
```
