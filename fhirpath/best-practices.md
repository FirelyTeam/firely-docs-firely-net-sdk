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
```

## The `%resource` and `%rootResource` variables

Some FhirPath statements use the `%resource` and `%rootResource` environment variables. When you evaluate an expression through the POCO extension methods,
the engine infers these automatically from the input: `%resource` becomes the resource that contains the node you are evaluating (or the root of the input
itself when it is not inside a resource), and `%rootResource` the outermost resource. So in the common case these variables just work, without you having to
configure anything:

```csharp
Patient p = new() {...}
var hasName = p.IsTrue("%resource.name.exists()");   // %resource is inferred from p
```

If you need `%resource` to point at a different resource than the one the engine would infer, set the `Resource` (and/or `RootResource`) property on a
`FhirEvaluationContext` explicitly — an explicitly-set value overrides the inferred one:

```csharp
var ctx = new FhirEvaluationContext { Resource = someResource.ToPocoNode(ModelInfo.ModelInspector) };
var hasName = p.IsTrue("%resource.name.exists()", ctx);
```

The engine can infer these variables because it works on a `PocoNode`, which navigates the POCO tree and keeps track of parent, contained, and `Bundle`
entry nodes — the same bookkeeping that makes `resolve()` work (see the previous section).

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
