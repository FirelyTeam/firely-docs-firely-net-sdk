# Best practices

Although it is seemingly easy to invoke FhirPath, there are a few details that are easy to get wrong.

## Start evaluation from the root

To make the `resolve()` function work well (e.g. to resolve to entries in a Bundle or to a contained resource), the FhirPath engine needs
to have "seen" all the resources while navigating through the data, which means you need to evaluate Bundles from their roots.

```csharp
Bundle b = new() {...}

// The engine has worked from the root of the bundle down, so it knows how to resolve to other entries
var active = b.Select("Bundle.entry.ofType(Patient).organization.resolve()");

// The engine was started from the nested Patient node, so it does not know how to find other entries.
var patient = b.Entry.Select(e => e.Resource).OfType<Patient>().First();
var active2 = patient.Select("organization.resolve()");

// This is fine too, since the context is transferred from call to call.
var org2 = b.Select("Bundle.entry.ofType(Patient)");
var active3 = org2.Select("organization.resolve()");
```

## Set `%resource` on the evaluation context

Although not many FhirPath statements use the `%resource` and `%rootResource` environment variables, they *do* get used, and the default constructors will
make it easy for you to *not* set them (blame us for that). To make sure these variables work well, you should pass a sensible `EvaluationContext` to the
FhirPath functions, even though they are optional:

```csharp
Patient p = new() {...}
var hasName = p.IsTrue("Patient.name.exists()",
    new FhirEvaluationContext { Resource = p.ToPocoNode(ModelInfo.ModelInspector) });
```

As you can see, we pass a new `FhirEvaluationContext` whose `Resource` is set to the root of the object. The FhirPath engine works on a `PocoNode`, which navigates
the POCO tree and keeps track of parent, contained, and `Bundle` entry nodes — the bookkeeping that makes `resolve()` work (see the previous section).

## Set the `ElementResolver` on the `FhirEvaluationContext`

Finally, the engine needs you to supply a delegate when you want `resolve()` to be able to reach out to instances of Resources (via uri) that it cannot locate itself.
The delegate takes a single string parameter (the uri) and returns a `PocoNode`:

```csharp
var ctx = new FhirEvaluationContext { Resource = p.ToPocoNode(ModelInfo.ModelInspector) };
ctx.ElementResolver = myResolver;

PocoNode? myResolver(string uri)
{
     var resolved = ...;
     return resolved.ToPocoNode(ModelInfo.ModelInspector);
}
```

## Set the `TerminologyService` on the `FhirEvaluationContext`

To use the FhirPath function `memberOf(valueset)`, you must set the `TerminologyService` property on the `FhirEvaluationContext`.
This gives the FhirPath engine a means to check whether a code is in a value set.

```csharp
var ctx = new FhirEvaluationContext
{
    TerminologyService = new LocalTerminologyService(resolver: ZipSource.CreateValidationSource())
};

var result = new Code("male").Scalar("memberOf('http://hl7.org/fhir/ValueSet/administrative-gender')", ctx);
```
