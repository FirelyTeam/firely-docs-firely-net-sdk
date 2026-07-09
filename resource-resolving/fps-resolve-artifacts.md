# How to resolve artifacts from packages

The most common use case for the FHIR Package source is to resolve conformance resources like `StructureDefinition`s, `ValueSet`s, and `CodeSystem`s from specific FHIR packages.
`FhirPackageSource` implements `IAsyncResourceResolver`, `IArtifactSource`, and `IConformanceSource`.
That means all conformance resources can be resolved by specifying their canonical url.

```csharp
FhirPackageSource resolver = new(ModelInfo.ModelInspector, new string[] { package1, package2 });

var patientProfile = (await resolver.TryResolveByCanonicalUriAsync("http://hl7.org/fhir/StructureDefinition/Patient")).Value as StructureDefinition;
var genderValueSet = (await resolver.TryResolveByCanonicalUriAsync("http://hl7.org/fhir/ValueSet/administrative-gender")).Value as ValueSet;
```

You can also find any resource by their type and id, or any file by their name or relative filepath.

```csharp
FhirPackageSource resolver = new(ModelInfo.ModelInspector, new string[] { package1, package2 });

//find resource by id
var pat = (await resolver.TryResolveByUriAsync("StructureDefinition/Patient")).Value as StructureDefinition;

//find file by name
Stream file = resolver.LoadArtifactByName("StructureDefinition-Patient.xml");

//find file by path
Stream file2 = resolver.LoadArtifactByPath("package/StructureDefinition-Patient.xml");
```

Next to that, there are some specific functions for certain resource types.

```csharp
FhirPackageSource resolver = new(ModelInfo.ModelInspector, new string[] { package1, package2 });

//Find CodeSystems by ValueSets
CodeSystem cs = resolver.FindCodeSystemByValueSet("http://hl7.org/fhir/ValueSet/address-type");

//Find ConceptMaps by source and/or target (sourceUri or targetUri can be null)
IEnumerable<ConceptMap> cms = resolver.FindConceptMaps(sourceUri: "http://hl7.org/fhir/ValueSet/data-absent-reason", targetUri: "http://hl7.org/fhir/ValueSet/v3-NullFlavor");

//Find NamingSystem by uniqueId
NamingSystem? ns = resolver.FindNamingSystem("http://snomed.info/sct");
```
