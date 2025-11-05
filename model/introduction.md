# Introduction to the FHIR Model in .NET

The `Hl7.Fhir.Model` namespace contains model classes that directly correspond to the published FHIR resources and data types, such as `Patient` and `HumanName`. These classes are generated using a [code generator](https://github.com/FirelyTeam/fhir-codegen) to ensure that the .NET implementation aligns precisely with the published FHIR specification.

```{note}
Only base resource classes are generated; profiles are not included.
```

The model classes are integral to the SDK and are used consistently throughout. For instance, they can be serialized to JSON and XML using the serializers, exchanged with FHIR servers via HTTP, and validated against profiles using the profile validator. These Plain Old CLR Objects (POCOs) are central to the SDK's functionality, making it essential to understand how to work with them.

This chapter provides guidance on using the model, including code examples. It features a complete example of creating and populating a `Patient` resource instance and concludes with a discussion of the `Bundle` resource type, accompanied by examples of working with Bundles.

To begin using the model, include the following `using` directive in your code:

```csharp
using Hl7.Fhir.Model;
```
