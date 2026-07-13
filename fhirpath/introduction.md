# Using FhirPath

The SDK contains a compiler and runtime for [FhirPath](http://hl7.org/fhirpath/).
FhirPath is an extraction and navigation language, and you can execute FhirPath expressions both on {ref}`FHIR POCO classes <FHIR-model>`
and on {ref}`ITypedElement-based data <elementmodel-intro>`. For both, the following (extension) methods are available:

| Method | Purpose |
|--------|---------|
| `Select()` | Returns the nodes produced by the expression. |
| `Scalar()` | Returns a single scalar value produced by the expression. |
| `Predicate()` | Returns true if the expression evaluates to `true` or `{}` (empty) and `false` otherwise. |
| `IsTrue()` | Returns true if the expression evaluates to `true` and `false` otherwise. |
| `IsBoolean()` | Determines whether the expression evaluates to a given boolean value. |

These functions exist as extension methods, so they can conveniently be called on either a POCO or `ITypedElement`:

```csharp
Patient p = new() {...}
var hasName = p.IsTrue("Patient.name.exists()");

ITypedElement ite = FhirXmlNode.Parse(...).ToTypedElement(...);
hasName = ite.IsTrue("Patient.name.exists()");
```

The `Select()` method will return a list of nodes - this is often used to navigate within the object and select a subset of the nodes within it:

```csharp
Patient p = new() {...}
var extensions = p.Select("Patient.extension");
```

When you run the `Select` method on a POCO, the nodes you will get returned are POCOs, in this example they would be of type `Hl7.Fhir.Model.Extension`.
When running FhirPath on `ITypedElement`, the nodes will be of type `ITypedElement` (and its `InstanceType` in this case would be `Extension`).

`Select()` consistently returns a list of nodes (an `IEnumerable<Base>`), even on a non-repeating element of a primitive type. Selecting a FHIR
element therefore yields the corresponding POCO: `p.Select("Patient.active")` returns a single node that is a `FhirBoolean`, not a string (true/false).

Note, however, that a *computed* expression produces a FhirPath system value rather than a FHIR POCO. For example, `p.Select("4+5")` returns a single
node holding the integer `9`, but that node is a FhirPath system value — it is not an `Hl7.Fhir.Model.Integer`. So when an expression can evaluate to
a computed value, don't assume the result is a FHIR type:

```csharp
Patient p = new() {...}

// Selecting a FHIR element yields the corresponding POCO
var active = p.Select("Patient.active").Single();   // a FhirBoolean

// A computed expression yields a FhirPath system value, not a FHIR POCO
var sum = p.Select("4+5").Single();                 // not an Hl7.Fhir.Model.Integer
```
