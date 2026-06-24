# Custom ClassMappings and PropertyMappings

The SDK's mapping infrastructure (`ClassMapping`, `PropertyMapping`, `ModelInspector`) is not limited to the types generated from the published FHIR specification. You can create custom mappings at runtime to represent resources and datatypes that are only known at runtime—for example, types derived from a StructureDefinition that is loaded dynamically.

## Shared vs. fresh ModelInspector

Before adding custom mappings you need to decide whether to extend the *shared* inspector or create a *fresh* one.

**The shared inspector** — `ModelInfo.ModelInspector` — is a singleton populated with all FHIR types for the release. It is the inspector used automatically by the convenience classes `FhirJsonSerializer`, `FhirXmlSerializer`, and `FhirClient` when you do not pass an explicit inspector. Adding custom `ClassMapping`s to the shared inspector means those types are immediately visible to all code that relies on those defaults, which can be convenient when you want the serializer and client to recognize your custom types without any extra configuration.

**A fresh inspector** is created by instantiating `ModelInspector` directly and importing the release assembly manually:

```csharp
var inspector = new ModelInspector(FhirRelease.R4);
inspector.Import(typeof(ModelInfo).Assembly);
```

This inspector is completely isolated. Custom mappings added to it do not affect the shared inspector or any other part of the application. You pass it explicitly to the lower-level `BaseFhirJsonSerializer` and `BaseFhirXmlDeserializer`:

```csharp
var json = new BaseFhirJsonSerializer(inspector).SerializeToString(resource);
var parsed = new BaseFhirJsonDeserializer(inspector).DeserializeResource(json);
```

Use a fresh inspector when you need isolation—for example, when a server hosts multiple custom resource profiles and each should be handled independently.

## Creating custom ClassMappings

You can create `ClassMapping` instances manually to represent resources or datatypes not known at compile time. Two built-in .NET types serve as the backing implementation:

- **`DynamicResource`**: for custom domain resources
- **`DynamicDataType`**: for custom complex datatypes

The example below defines a custom datatype and a custom resource, registers them with a fresh `ModelInspector`, and round-trips an instance through JSON:

```csharp
// Start from a fresh inspector populated with the standard R4 types.
var inspector = new ModelInspector(FhirRelease.R4);
inspector.Import(typeof(ModelInfo).Assembly);

// Look up primitive ClassMappings to use as property types.
var stringMapping = inspector.FindClassMapping("string")!;
var integerMapping = inspector.FindClassMapping("integer")!;

// Define a custom complex datatype 'MyMeasurement' with two elements.
var measurementMapping = new ClassMapping(inspector, "MyMeasurement", typeof(DynamicDataType), dt =>
[
    new PropertyMapping(dt, "unit", typeof(FhirString), [stringMapping]) { Order = 10 },
    new PropertyMapping(dt, "score", typeof(Integer), [integerMapping]) { Order = 20 }
])
{
    Canonical = "http://example.org/fhir/StructureDefinition/MyMeasurement"
};

// Define a custom resource 'MyObservation' with a choice element 'value'
// that can be either a string or a MyMeasurement.
var observationMapping = new ClassMapping(inspector, "MyObservation", typeof(DynamicResource), r =>
[
    new PropertyMapping(r, "subject", typeof(FhirString), [stringMapping]) { Order = 10 },
    new PropertyMapping(r, "value", typeof(DataType), [stringMapping, measurementMapping]) { Order = 20 }
])
{
    Canonical = "http://example.org/fhir/StructureDefinition/MyObservation"
};

// Register the custom mappings with the inspector.
inspector.ClassMappings.Add(measurementMapping);
inspector.ClassMappings.Add(observationMapping);

// CreateInstance() sets DynamicTypeName automatically from the mapping name.
var measurement = (DynamicDataType)measurementMapping.CreateInstance();
measurement.SetValue("unit", new FhirString("kg"));
measurement.SetValue("score", new Integer(75));

var observation = (DynamicResource)observationMapping.CreateInstance();
observation.SetValue("subject", new FhirString("patient-1"));
observation.SetValue("value", measurement);

// Serialize and deserialize using the custom inspector.
var json = new BaseFhirJsonSerializer(inspector).SerializeToString(observation);
var parsed = new BaseFhirJsonDeserializer(inspector).DeserializeResource(json);
```

## Creating custom PropertyMappings

You can also add custom `PropertyMapping` instances to an *existing* `ClassMapping`. This is useful when a resource carries dynamic elements stored in the overflow (see [dynamic features](dynamic-features.md)) and you want the serialization infrastructure to handle them with the correct types, including choice and repeating elements.

The example below adds three custom properties to the standard `Patient` mapping so that three overflow elements are serialized and deserialized with the intended types:

```csharp
var patientMapping = inspector.FindClassMapping(typeof(Patient))!;

// Create custom PropertyMappings for new element `patientLocation` of type `FhirUri`. This ensures that when deserializing,
// the overflow will contain a `FhirUri` instance for this element.
var customPropertyA = new PropertyMapping(patientMapping, "patientLocation", typeof(FhirUri));
patientMapping.PropertyMappings.Add(customPropertyA);

// Create custom PropertyMapping for new choice element `remarks` of type `FhirString` or `Markdown`. This will ensure that
// when (de)serializing, the element will be recognized as a choice type, and contain the type suffix.
var customPropertyB = new PropertyMapping(patientMapping, "remarks", typeof(DataType), [typeof(FhirString), typeof(Markdown)]);
patientMapping.PropertyMappings.Add(customPropertyB);

// Create custom PropertyMapping for new element `newList` of type `List<FhirString>`.
// This will ensure that when (de)serializing, the element will be treated as a repeating element.
var customPropertyC = new PropertyMapping(patientMapping, "newList", typeof(List<FhirString>));
patientMapping.PropertyMappings.Add(customPropertyC);
```
