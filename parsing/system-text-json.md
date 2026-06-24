(system-text-json)=
# Using FHIR with System.Text.Json

The SDK plugs into `System.Text.Json`, so you can read and write FHIR POCOs with the standard `JsonSerializer`. This is the natural choice when FHIR types are part of a larger object graph, or when your application already uses `System.Text.Json` (for example in ASP.NET Core).

```{note}
This integration is JSON only. For XML, use the {doc}`serialization` / {doc}`deserialization` classes.
```

## Setting up the options

`ForFhir()` registers the FHIR converter on a `JsonSerializerOptions`. Without arguments it targets your project's FHIR version; pass a `ModelInspector` for a custom model:

```csharp
var options = new JsonSerializerOptions().ForFhir();
```

```{important}
Reuse a single `JsonSerializerOptions` instance across calls. Building a new one each time degrades performance dramatically.
```

## Reading and writing

Once the options are set up, use `JsonSerializer` as usual:

```csharp
var patient = JsonSerializer.Deserialize<Patient>(json, options);
string json2 = JsonSerializer.Serialize(patient, options);
```

Add `.Pretty()` (or `.Compact()`) for indented output:

```csharp
var options = new JsonSerializerOptions().ForFhir().Pretty();
```

## Errors and modes

Deserialization throws a `DeserializationFailedException` on problems, exactly like the deserializer classes. The same modes and fine-tuning apply, configured fluently on the options:

```csharp
var options = new JsonSerializerOptions().ForFhir().UsingMode(DeserializationMode.Recoverable);
```

`.Enforcing(...)` and `.Ignoring(...)` work here too. See {doc}`error-handling` for the modes and what they mean.

## Summaries

To serialize a summary, set a filter on a `FhirJsonConverterOptions` and pass it to `ForFhir`:

```csharp
var converterOptions = new FhirJsonConverterOptions
{
    SummaryFilterFactory = () => SerializationFilter.ForSummary()
};
var options = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector, converterOptions);
```

The available filters are listed under {doc}`serialization`.

```{note}
As with the other serializers, no validation is performed on serialization — an invalid POCO produces invalid FHIR. {ref}`Validate <validation>` first if you need guaranteed-valid output.
```

## CDS Hooks

An experimental `ForCdsHooks()` variant configures the options for [CDS Hooks](https://cds-hooks.hl7.org/) messages (which embed FHIR). It is subject to change and should be used with care.
