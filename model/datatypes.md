# FHIR Data Types

In FHIR, the [data types](http://www.hl7.org/fhir/datatypes.html) are categorized into 'primitive' and 'complex' data types.

(datatypes-primitives)=
## FHIR Primitives

FHIR provides a comprehensive set of [primitive data types](https://hl7.org/fhir/datatypes.html#primitive), such as `string`, `integer`, `boolean`, etc.

Since every data type in FHIR can be [extended](http://www.hl7.org/fhir/extensibility.html), FHIR primitives are not truly atomic (as they are in .NET). To accommodate this, the SDK includes a dedicated FHIR class for each FHIR primitive type.

The class names match the FHIR type names, except when a conflict arises with existing .NET data types. In such cases, the class name is prefixed with "Fhir". For example, a FHIR `string` becomes `FhirString` in .NET.

For properties representing elements with primitive data types, the SDK provides *two* properties in the class.

### The "Helper" Property
One property shares the same name as the corresponding element, such as `Active` in the `Patient` class. This property uses the standard .NET data type:

```csharp
var pat = new Patient();
pat.Active = true;
```

### The "Element" Property
The other property appends the suffix 'Element' to the name, such as `ActiveElement`. This property is used when you need access to the element's `ElementId` or `Extension`:

```csharp
pat.ActiveElement = new FhirBoolean(true);
pat.ActiveElement.ElementId = "patActive";
pat.ActiveElement.Extension.Add(...);
pat.ActiveElement.Value.Should().Be(true);
```

The helper property automatically creates a new instance of the data type for the element property and sets the `Value` property. This allows you to combine both styles:

```csharp
pat.Active = true;
pat.ActiveElement.Extensions.Add(...);
```

## Complex Data Types

Complex data types in FHIR group related values together, such as `Address`, `Identifier`, and `Quantity`. The [FHIR specification](http://www.hl7.org/fhir/datatypes.html) defines the elements included in these data types.

The SDK provides classes for each complex data type, with properties for all their elements. Most elements are of primitive data types, but some may also be complex types.

Working with complex data types is similar to working with resources and primitive data types. You can also use C# initializer syntax for convenience:

```csharp
var id = new Identifier()
{
  System = "http://hl7.org/fhir/sid/us-ssn",
  Value = "000-12-3456"
};
```

## Shared Data Types

To simplify working with multiple FHIR versions, all *normative* data types are part of a "shared" set. These shared types remain consistent (and originate from the same assembly) across FHIR versions. For most use cases, this is seamless. However, some minor changes exist between versions, such as in the `Attachment` data type.

```{note}
The term "normative" refers to parts of the specification that are stable and will not undergo breaking changes. While this does not guarantee no breaking changes in POCOs, it ensures these classes are reusable across different FHIR versions and NuGet packages.
```

For instance, the `size` element in `Attachment` changed from an unsigned int in STU3 to a 64-bit integer in R5. Since there is only one shared class for `Attachment`, the `size` element is represented as a `PrimitiveType`. Similar to [choice elements](choice-elements), you can assign the appropriate value based on the FHIR version. IntelliSense also indicates the correct type to use for each FHIR version:

```csharp
var att = new Attachment();
att.SizeElement = new Integer64();   // in R5
att.SizeElement = new UnsignedInt();  // in STU3
att.SizeElement = new FhirBoolean();  // incorrect
```

Although it is possible to assign an incorrect data type to `SizeElement`, validation (performed in the context of a specific FHIR version) will catch such errors.

Since `size` is a primitive, it has helper properties—one for the "current" (R5) version and another for STU3. The older version is explicitly suffixed with its type name:

```csharp
att.SizeUnsignedInt = 1000;
att.Size = 1000;
```

```{note}
There are two sets of shared data types:
* Types shared across STU3, R4, and newer releases, found in the `Hl7.Fhir.Base` assembly. These include common types like `FhirBoolean`, `Code`, and `HumanName`.
* Types shared across R4 and newer releases (with STU3 having its own incompatible set). These are included in the `HL7.Fhir.Conformance` assembly and include types like `CodeSystem`, `ElementDefinition`, and `ValueSet`.

This split exists because the latter set was not normative in STU3 and underwent incompatible changes, preventing sharing across versions.
```

## Code and Code&lt;T&gt;
TODO