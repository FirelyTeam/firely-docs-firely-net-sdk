# Choice elements

In the FHIR specification, some resource elements are defined as "choice elements." These allow you to select the type of value to assign to the element from a predefined list of possible types.

For example, the `deceased[x]` element in the `Patient` resource is a choice element, as indicated by the `[x]` suffix. The specification also lists the allowed types for the element:

```{image} ../images/fhir_patient_deceased.png
```

In the SDK, the corresponding property is of type `DataType`, which serves as the base class for all complex and primitive data types. The SDK enhances the generated property with attributes that specify the allowed data types. These attributes are used during validation to ensure that only permissible types are assigned:

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

This means that in your code, you must first create an instance of the desired data type before assigning it to the field. For instance, if you want to use a date for the `Deceased` field of a `Patient`, you could implement it as follows:

```csharp
pat.Deceased = new FhirDateTime("2015-04-23");   // Assigning a date
pat.Deceased = new FhirBoolean(true);           // Assigning a boolean
```

To access the property, it is recommended to use a `switch` statement:

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
