(deserialization)=
# Deserialization

Deserialization turns serialized FHIR — a string, stream, or reader — into the SDK's POCO objects. All the types below live in:

```csharp
using Hl7.Fhir.Serialization;
```

## Choosing a deserializer

For a single FHIR version (the usual case), use `FhirJsonDeserializer` or `FhirXmlDeserializer`. These are wired to the version you referenced through your NuGet package, so they need no configuration:

```csharp
var patient = FhirJsonDeserializer.DEFAULT.DeserializeResource(json);
```

If you work with a custom or runtime-chosen model, use the `BaseFhirJsonDeserializer` / `BaseFhirXmlDeserializer` base classes instead and pass a `ModelInspector` (see {doc}`the ModelInspector </model/model-metadata>`):

```csharp
var deserializer = new BaseFhirJsonDeserializer(inspector);
```

`DEFAULT` is one of several ready-made presets; the others control how strict the deserializer is. See {doc}`error-handling` for those and for how problems are reported.

## Reading a resource

`DeserializeResource` reads a complete resource. Use it when you do not know the resource type up front:

```csharp
Resource resource = FhirJsonDeserializer.DEFAULT.DeserializeResource(json);
if (resource is Patient patient)
    Console.WriteLine(patient.Active);
```

When you do know the type, `Deserialize<T>` returns it directly:

```csharp
var patient = FhirJsonDeserializer.DEFAULT.Deserialize<Patient>(json);
```

## Reading a datatype

To read a fragment that is not a resource — a `HumanName`, say — use `Deserialize<T>` with that type (or `DeserializeObject` when the type is only known at runtime):

```csharp
var name = FhirJsonDeserializer.DEFAULT.Deserialize<HumanName>(json);
```

## Reading from a reader

Every method also accepts a reader instead of a string, which avoids loading the whole document into memory — a `Utf8JsonReader` for JSON, an `XmlReader` for XML:

```csharp
var patient = FhirJsonDeserializer.DEFAULT.Deserialize<Patient>(ref reader);
```

## XML

Everything above applies to XML through the matching `FhirXmlDeserializer` / `BaseFhirXmlDeserializer`; only the class and reader type change:

```csharp
var patient = FhirXmlDeserializer.DEFAULT.Deserialize<Patient>(xml);
```

## Handling invalid input

The methods shown here throw a `DeserializationFailedException` when the input has problems. To collect issues without throwing, use the `TryDeserialize…` variants, and to control which problems are treated as errors, choose a mode. Both are covered in {doc}`error-handling`.
