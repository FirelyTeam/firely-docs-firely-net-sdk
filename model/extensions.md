# Extensions

FHIR provides a [powerful extension mechanism](https://hl7.org/fhir/extensibility.html) to include data that is not part of the standard specification. This mechanism allows implementers to add context-specific data without compromising interoperability with other systems. Extensions are widely used in FHIR, and many standard extensions are defined in the FHIR specification.

In the .NET SDK, extensions are represented by the `Extension` class. Every `Element` (the base class for all FHIR data types) includes an `Extension` property, which is a list of extensions. 

```{note}
The FHIR specification has specific rules about which elements and data types can be extended. These rules are implemented in the SDK using the `IExtendable` and `IModifierExtendable` interfaces. For instance, `Bundle` cannot be extended at all, primitives can have extensions but not modifier extensions, while `BackboneElement` and `DomainResource` can support both types of extensions.
```

```{warning}
The properties `Extension.Url` and `Element.ElementId` need some extra care as they should not be extended: they are serialized as attributes in the XML representation. Before SDK6, we reflected this fact in the model, but since then, we have removed these constructs to simplify working with the POCOs.
```

The set of extensions that FHIR provides is not fixed. HL7 has defined a "core" set of widely recognized extensions, but additional extensions may be defined in (national) implementation guides. The most comprehensive list of core extensions is available in the [extension registry](https://hl7.org/fhir/extensions/extension-registry.html).

The following code example demonstrates how to add a few extensions to a `Patient` resource:

- [Birthplace](https://hl7.org/fhir/extensions/StructureDefinition-patient-birthPlace.html) (of type `Address`)
- [Time of birth](https://hl7.org/fhir/extensions/StructureDefinition-patient-birthTime.html) (of type `dateTime`)
- [Mother's maiden name](https://hl7.org/fhir/extensions/StructureDefinition-patient-mothersMaidenName.html) (of type `string`)

```csharp
var birthAddress = new Address() { City = "Seattle" };
pat.AddExtension("http://hl7.org/fhir/StructureDefinition/birthPlace", birthAddress);

pat.BirthDate = "1983-04-23";
pat.BirthDateElement.AddExtension("http://hl7.org/fhir/StructureDefinition/patient-birthTime", 
  new FhirDateTime(1983, 4, 23, 7, 44));
pat.SetString("http://hl7.org/fhir/StructureDefinition/patient-mothersMaidenName", "Kramer");
```

Note that the `birthTime` extension is specifically added to the `BirthDate` property (as specified in the extension definition), not to the `Patient` resource itself. Additionally, for a few simple primitive types (e.g., `string`, `boolean`, `integer`), the SDK provides direct `SetXXXX()` methods to simplify setting an extension.