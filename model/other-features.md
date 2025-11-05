# Other Features of the POCOs

## Fluent Initializers

The SDK provides additional initialization methods for several data types. These methods are designed to simplify object construction using fluent notation. Visual Studio's IntelliSense can guide you through the available options as you type. Alternatively, you can explore the `Hl7.Fhir.Model` namespace in the Object Browser to review the methods and their attributes.

For example, the `HumanName` data type includes methods that allow you to construct a name in a single statement:

```csharp
pat.Name.Add(new HumanName().WithGiven("Christopher").WithGiven("C.H.").AndFamily("Parks"));
```

If you need to populate fields beyond `Given` and `Family`, you can first create a `HumanName` instance using this approach and then add additional details later. Alternatively, you can skip the fluent notation and populate all fields as described in other sections.

## Annotations

All POCOs in the SDK support annotations, which allow you to attach arbitrary data to a POCO instance without modifying its class definition. This feature is particularly useful for storing metadata or other information relevant to the instance or its processing, but not part of the FHIR specification:

```csharp
var patient = new Patient();
patient.AddAnnotation(new MyCustomMetadata { ProcessedBy = "SystemX", Timestamp = DateTime.UtcNow });

if (patient.TryGetAnnotation<MyCustomMetadata>(out var metadata))
{
        Console.WriteLine($"Patient processed by {metadata.ProcessedBy} at {metadata.Timestamp}");
}
```

## The `IIdentifiable<T>` Interface

FHIR resources with an `Identifier` element have corresponding POCOs that implement the `IIdentifiable<T>` interface, where `T` represents the type of the `Identifier` property. In practice, `T` is either `Identifier` or `List<Identifier>`. The `IIdentifiable<T>` interface inherits from the marker interface `IIdentifiable`.

Extension methods such as `GetIdentifier()` and `TryGetIdentifier()` are available to quickly retrieve an identifier by its system from a POCO implementing `IIdentifiable`.

## The `ICode<T>` Interface
```{note}
This is an advanced feature that is mostly used for model binding within the CQL engine, but can also be useful in other scenarios.
```

The `ICode<T>` interface is used to represent the "primary code" element of a resource, as defined by the FHIR model for Clinical Quality Language (CQL). While this element is often named "Code," it may have a different name depending on the POCO design. The SDK uses the CQL binding to identify these elements.

Resources with a primary code implement `ICode<T>`, where `T` can be one of the following types:

- `Code`
- `CodeableConcept`
- `List<CodeableConcept>`
- `FhirString`
- `DataType`

The `DataType` type (an abstract class) is used when the coded property is a choice, such as elements related to Medication that can be either `Coding` or `Reference`. Although the current set of types is limited, future changes to the CQL binding may expand this list. Implementers should be prepared to handle all coded types (`Code`, `Coding`, `CodeableConcept`, `Quantity`) and data types (`string`, `uri`).

The `ICode<T>` interface extends the `ICode` interface, which includes a single method, `ToCodings()`. This method returns the coded contents as a list of `Coding` instances. The conversion of coded types to `Coding` follows the guidelines outlined in the [FHIR Terminologies documentation](https://hl7.org/fhir/terminologies.html#4.1).

