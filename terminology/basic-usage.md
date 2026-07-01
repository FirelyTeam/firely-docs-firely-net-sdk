# A basic example

The quickest way to try a terminology service is the `LocalTerminologyService`, which works entirely in-process against a set of FHIR conformance resources. Here we validate that a code is a member of a value set:

```csharp
var resolver = ZipSource.CreateValidationSource();
var service = new LocalTerminologyService(resolver);

var parameters = new ValidateCodeParameters()
    .WithValueSet("http://hl7.org/fhir/ValueSet/administrative-gender")
    .WithCoding(new Coding("http://hl7.org/fhir/administrative-gender", "male"));

Parameters result = await service.ValueSetValidateCode(parameters);
```

Every operation follows the same shape: build its input with the matching helper class (`ValidateCodeParameters` here), call the operation, and read the output from the returned `Parameters` — for validate-code, a `result` boolean and an explanatory `message`.

To reach operations the local service does not support, or to use a dedicated terminology server, use the `ExternalTerminologyService`, which sends each operation to a FHIR endpoint. For example, expanding a value set:

```csharp
var client = new FhirClient("https://someterminologyserver.org/fhir");
var service = new ExternalTerminologyService(client);

var parameters = new ExpandParameters()
    .WithValueSet(url: "http://snomed.info/sct?fhir_vs=refset/142321000036106")
    .WithFilter("met")
    .WithPaging(count: 10);

var expanded = await service.Expand(parameters) as ValueSet;
```

See {doc}`implementations` for what each service supports, and {doc}`composing-services` for combining a local and an external service.
