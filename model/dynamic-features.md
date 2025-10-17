# Dynamic Access to data

Although accessing `POCO`s using properties is the most common way to work with FHIR data, the SDK provides several features and methods for dynamic access. These are useful when the data structure is not known at compile time, when building generic tools that operate on FHIR data, or when you must work with invalid data that does not conform to the FHIR specification or that contains unknown elements (for example, from future FHIR versions).

## TryGetValue/SetValue

You use TryGetValue() to retrieve the value of an element by its name. Note that this is the name used in the FHIR specification, not the C# property name. For example, to get the value of the active element on a Patient resource, use:

```csharp
if (patient.TryGetValue("active", out var activeElement))
{
    var isActive = activeElement.Value;
    isActive.Should().Be(patient.Active);
}
```

To set the value of an element, use `SetValue()`:

```csharp
patient.SetValue("active", new FhirBoolean(true));
```

`TryGetValue()` will return `false` when an element is `null` or an empty list, and `true` otherwise. `SetValue()` can also be used to set elements to `null` or to an empty list.

The SDK also implements indexers that internally call `TryGetValue()` and `SetValue()`, allowing a more concise syntax:

```csharp
var isActive = patient["active"]?.Value;
isActive.Should().Be(patient.Active);
```

```{note}
These dynamic access methods work for all elements, but not for the primitive's `Value` property. For example, you cannot use
`TryGetValue("value", out var x)` to get the value of a `FhirBoolean`.
```

## Overflow

`SetValue()` lets you set elements that are not defined in the FHIR specification for that resource, or set existing elements to invalid values. Data that the generated `POCO`s do not have properties for is stored in the *overflow*. The main use cases for overflow are working with future FHIR versions that introduce new elements or change types/cardinalities, or dealing with invalid data that does not conform to the FHIR specification.

```csharp
// Setting an unknown element (e.g. from a future FHIR version)
patient.SetValue("mothersMaidenName", new FhirString("some value"));
patient["mothersMaidenName"].Value.Should().Be("some value");
```

Setting an existing element to an invalid value will make it impossible to use the generated property, but the value can still be retrieved using `TryGetValue()` or the indexer:

```csharp
patient.SetValue("active", new FhirString("not a boolean"));
patient["active"]?.Value.Should().BeOfType<FhirString>();

// These will throw
Console.WriteLine(patient.Active);
Console.WriteLine(patient.ActiveElement);
```

To check whether a `POCO` has overflow data, use the `HasOverflow` property.

## JsonValue

Like any `POCO`, the primitive datatypes such as `FhirString`, `FhirBoolean`, etc., support overflow as well. Primitives also expose a `JsonValue` property that allows getting or setting the raw JSON value of the primitive. This is useful when working with invalid primitive data.

```csharp
var fhirString = new FhirString();
fhirString.JsonValue = true;

var invalidCode = new Code<AdministrativeGender>();
invalidCode.JsonValue = "fmale";
```

Normally, `JsonValue` contains the same .NET type as used for the literal in the FHIR JSON representation:

| Datatype         | JsonValue (.NET system type) |
|------------------|------------------------------|
| `FhirString`     | `string`                     |
| `FhirBoolean`    | `bool`                       |
| `Integer`        | `int`                        |
| `PositiveInt`    | `int`                        |
| `UnsignedInt`    | `int`                        |
| `FhirDecimal`    | `decimal`                    |
| `Code<T>`        | `string`                     |
| `Instant`        | `string`                     |
| `Base64Binary`   | `string`                     |
| *all others*     | `string`                     |

Attempting to access the strongly typed `Value` property of a primitive with an invalid value will throw an exception. Use `ValidateObjectValue()` to check whether the current `JsonValue` is valid; see the chapter on [POCO validation](../validation/poco-validation) for more information.

```{note}
The serializers and deserializers fully support working with overflow and `JsonValue`. Together these features enable preserving invalid data for round-tripping and provide support for future (or incompatible) FHIR versions. The `ModelInspector` is also involved here: it determines which properties are (de)serialized and how. See the documentation on [metadata](model-metadata) and on how it affects [serialization](../parsing).
```

```{note}
The `JsonValue` property was introduced in SDK 6 and supersedes the existing `ObjectValue` property, which provided similar functionality but used different concrete types. The differences were significant enough that a new property was introduced rather than changing the existing one.
```

## EnumerateElements

The `EnumerateElements` method lets you iterate over all elements in a resource, including overflow. 

```{note}
As with `TryGetValue()`, `EnumerateElements()` does not enumerate the `Value` elements of the primitives.
```

The method returns an `IEnumerable<KeyValuePair<string, object>>`, where the key is the element name and the value is either a `Base` instance or a `List<T>` (where `T` is a FHIR datatype). Elements that are `null` or empty lists are not included in the enumeration.

It is often easiest to handle the returned elements with pattern matching:

```csharp
foreach (var element in patient.EnumerateElements())
{
    switch (element.Value)
    {
        case Base baseElement:
            // Process complex element
            break;
        case IReadOnlyList<Base> fhirElements:
            // Process list of FHIR elements
            break;
        default:
            // Cannot happen.
            break;
    }
}
```

```{note}
We use `IReadOnlyList<Base>` to match lists of FHIR elements, not `List<T>`. Matching directly against `List<T>` will not work because the concrete list type is not known at compile time; the covariance provided by `IReadOnlyList<T>` is required.
``` 

## Legacy dynamic access methods

Before SDK 6, the SDK offered several different ways to access FHIR data dynamically:

* Call `Children()` to get all child elements.
* Use the `IReadOnlyDictionary<string, object>` interface implemented by `Base` and `Resource` to get or set element values by name.
* Use `ITypedElement` to navigate the FHIR data tree dynamically.

These legacy methods are still available for backward compatibility, but we recommend using the new methods described above for new development. See the sections below for guidance on migrating from these legacy approaches.

### Children()

The `Children()` method returns an `IEnumerable<Base>` with all child elements. Note that `Children()` flattened lists and returned strings for `Narrative.div` and `Element.id`, whereas `EnumerateElements()` returns the corresponding FHIR model instances (`XHtml` and `FhirUri`, respectively).

### IReadOnlyDictionary

Previously, `POCO`s implemented `IReadOnlyDictionary<string, object>`, but this caused issues in many development tools, debugger viewers, and serializers. The new dynamic access methods align closely with the `IReadOnlyDictionary` however, so transitioning should be easy. You can also call `ToPocoNode()` on any `Base` instance; `PocoNode` implements `IReadOnlyDictionary`. See the chapter on [PocoNode navigation](poconode-navigation) for more information.

### ITypedElement

The `ITypedElement` interface provides a way to navigate the FHIR data tree dynamically. The new dynamic methods are not a drop-in replacement because `ITypedElement` represents a more generic data model that is not tied to the `POCO`s. To enable gradual migration, we still support conversion between `Base` and `ITypedElement` using the `ToTypedElement()` and `ToPoco()` extension methods. As with `IReadOnlyDictionary`, we support this legacy scenario through the new [PocoNode navigation](poconode-navigation). 