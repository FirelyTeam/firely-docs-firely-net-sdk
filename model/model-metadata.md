# FHIR model metadata

The C# classes that represent FHIR resources and datatypes are generated from the metadata in the published FHIR specification. Generally, each FHIR datatype or resource maps to a single C# class, and each element becomes a property on that class. Important `ValueSet`s from the specification are converted to C# enumerations. The classes and properties carry additional metadata expressed as attributes—information such as cardinality, allowed datatype choices for polymorphic properties, and so on. Here is a fragment of a generated class:

```csharp
[FhirType("Patient","http://hl7.org/fhir/StructureDefinition/Patient")]
public partial class Patient : Hl7.Fhir.Model.DomainResource
{
    /// <summary>
    /// FHIR Type Name
    /// </summary>
    public override string TypeName { get { return "Patient"; } }
```

This declaration illustrates a few points:

- This is a C# class called `Patient`, which represents the FHIR resource `Patient`.
- The attribute specifies the canonical URL for the FHIR definition of this resource.
- The class derives from `DomainResource` (see also [FHIR resource hierarchy](model-overview)), indicating that it is a `Resource`.

Each datatype also exposes a `TypeName` property containing the FHIR type name (the same value used in the `FhirType` attribute, but easier to access).

Properties include additional metadata, for example:

```csharp
/// <summary>
/// Time of assessment
/// </summary>
[FhirElement("effective", InSummary=true, Order=150, Choice=ChoiceType.DatatypeChoice)]
[CLSCompliant(false)]
[AllowedTypes(typeof(Hl7.Fhir.Model.FhirDateTime),typeof(Hl7.Fhir.Model.Period))]
[DataMember]
public Hl7.Fhir.Model.DataType? Effective
{
```

This example shows an element named `effective`. The attributes indicate this element is included "in summary" (a FHIR feature for concise search results), and that it is a *choice* element: it can be either a FHIR `dateTime` or a FHIR `period` (represented here by the corresponding C# types). Other attributes describe cardinality and additional constraints.

You can of course access these attributes using `System.Reflection`, but the SDK needs this metadata frequently and therefore provides cached, optimized helpers: `ClassMapping`, `PropertyMapping`, and `ModelInspector`. These types make it easy and efficient to retrieve the metadata encoded in the attributes. We describe those classes below.

In the section on [datatypes](datatypes), we discussed that shared datatypes, such as `Resource`, `HumanName`, and `Identifier` are distributed in the shared assemblies `Hl7.Fhir.Base` and `Hl7.Fhir.Conformance`. As a result, when working with a single FHIR release you will typically use three assemblies: the two shared assemblies and the release-specific assembly that contains types that vary between releases.

## The ModelInspector

The SDK's `ModelInspector` represents the metadata (in the form of `ClassMapping` instances) for the datatypes and resources of a single FHIR release. You can retrieve a `ClassMapping` by FHIR name, by .NET type, or by its canonical URL (the URL assigned to the FHIR type, usually `http://hl7.org/fhir/StructureDefinition/<typename>`). For multiple FHIR releases, you would use one `ModelInspector` per release.

You can create a `ModelInspector` and import assemblies manually, but this is usually unnecessary. You can use the static `ModelInfo.ModelInspector` property or call `ModelInspector.ForAssembly()` with a reference to the assembly that contains the POCO classes for the release:

```csharp
var inspector = ModelInfo.ModelInspector;
// Equivalent: var inspector = ModelInspector.ForAssembly(typeof(ModelInfo).Assembly);
var someDatatypeMapping = inspector.FindClassMapping("HumanName");
var aResourceMapping = inspector.FindClassMapping(typeof(Observation));
var anotherResource = inspector.FindClassMappingByCanonical("http://hl7.org/fhir/StructureDefinition/Procedure");
```

When working with multiple versions, you can use `external alias` to keep each `ModelInfo` reference distinct. You can also construct your own `ModelInspector` and add specific assemblies dynamically.

Calling `ModelInfo.ModelInspector` or `ModelInspector.ForAssembly()` repeatedly for the same assembly returns the same `ModelInspector` instance, so it is cheap and safe to do so.

## The ClassMapping

A `ClassMapping` contains the metadata derived from the `FhirTypeAttribute` and lists the type's properties via `PropertyMapping` instances. When you obtain `ClassMapping`s through `ModelInspector` you typically do not need to construct them yourself. Each `ClassMapping` also maintains a reference to its parent `ModelInspector`, allowing you to navigate back to the inspector if you need related metadata.

```{note}
At this moment, there is little need to create your own `ClassMappings`, but in a future release we may add functionality to create custom mappings at runtime, for example to represent resources that were unknown at compile time.
```

## The PropertyMapping

The `PropertyMapping` contains metadata from the `FhirElementAttribute`, such as the element name, whether it is "in summary", and its serialization order. You can access the underlying `PropertyInfo` via the `NativeProperty` property if you need reflection-level details.

There are three common ways to access properties on a `ClassMapping`:

- The `PropertyMappings` collection returns all properties that represent the FHIR elements of the type.
- `FindMappedElementByName()` finds a `PropertyMapping` by the exact element name.
- `FindMappedElementByChoiceName()` also supports choice elements: for example, passing `onsetDateTime` will return the `PropertyMapping` for the `onset` element.

It is possible to create custom `PropertyMapping` instances. This can be useful when building custom `ClassMapping`s or when providing metadata for dynamic elements stored in the overflow (see [dynamic features](dynamic-features.md)). Supplying type information for such elements enables the serialization and deserialization infrastructure to handle them correctly, including correct treatment of choice and repeating elements.

The example below shows how to add three custom properties to the existing `Patient` mapping so that three dynamic elements stored in the overflow will be serialized and deserialized with the intended types:

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

