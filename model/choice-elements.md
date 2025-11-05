# Choice Elements

In the FHIR specification, certain resource elements are defined as "choice elements." These elements allow you to assign a value from a predefined set of possible types.

For example, the `deceased[x]` element in the `Patient` resource is a choice element, as indicated by the `[x]` suffix. The specification also defines the allowed types for this element:

![Example with deceased[x]](../images/fhir_patient_deceased.png)

In the SDK, the corresponding property is represented as a `DataType`, which is the base class for all complex and primitive data types. The SDK augments the generated property with attributes that specify the permissible data types. These attributes are used during validation to ensure that only valid types are assigned:

```csharp
/// <summary>
/// Indicates if the individual is deceased or not.
/// </summary>
[FhirElement("deceased", InSummary=true, IsModifier=true, Order=150, Choice=ChoiceType.DatatypeChoice)]
[AllowedTypes(typeof(Hl7.Fhir.Model.FhirBoolean), typeof(Hl7.Fhir.Model.FhirDateTime))]
[DataMember]
public Hl7.Fhir.Model.DataType? Deceased
{
  // Implementation of property
}
```

This means that in your code, you must first create an instance of the appropriate data type before assigning it to the property. For example, to use a date for the `Deceased` field of a `Patient`, you could write:

```csharp
pat.Deceased = new FhirDateTime("2015-04-23");   // Assigning a date
pat.Deceased = new FhirBoolean(true);           // Assigning a boolean
```

To access the property, it is recommended to use a `switch` statement to handle the different possible types:

```csharp
switch (pat.Deceased)
{
  case FhirDateTime dateTime:
    // Handle date values
    break;
  case FhirBoolean boolean:
    // Handle boolean values
    break;
  default:
    // Handle unexpected types, possibly from another FHIR version
    break;
}
```
