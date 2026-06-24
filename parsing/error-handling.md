(deserialization-errors)=
# Error handling and deserialization modes

When you deserialize FHIR data, the input may not be perfectly valid: it could come from an older or newer FHIR version, from a non-conformant system, or simply contain mistakes. This page explains how the SDK 6 deserializers report such problems, and how you can control which problems are treated as errors.

## What the deserializer checks (and what it does not)

In SDK 6 the deserializer has one overriding goal: **get all the data from the input into the POCO**. Anything that fits a typed property is placed there; anything that does not fit — an unknown element, a value of the wrong type, a code outside the value set — is still captured, in the *overflow* of the object (see {doc}`dynamic features </model/dynamic-features>`). Because no data is silently dropped, the deserializer can afford to be lenient and simply *report* what was irregular rather than refuse to continue.

```{note}
This is a deliberate change from pre-SDK 6 behaviour. Older parsers reported an error for anything that did not map neatly onto the POCO, because there was nowhere else for the data to go. Now that overflow exists, the deserializer's job is narrowed to reporting genuine syntax and structural problems, while still letting you recover the data.
```

The issues a deserializer can raise come from two distinct phases:

- **Syntax and structure** — problems with the FHIR JSON or XML format itself, raised by the parser. These are reported as `FhirJsonException` (codes `JSON…`) or `FhirXmlException` (codes `XML…`).
- **Model validation** — problems with the *content* relative to the model, such as a value outside an enumeration or a violated cardinality. These are raised by a validator that runs *during* parsing (see [Validation during parsing](#validation-during-parsing) below) and are reported as `CodedValidationException` (codes `PVAL…`).

A full reference of every code and the exact behaviour of the deserializer for each is in {doc}`deserialization-behavior`.

## How errors are reported

Each deserializer offers two styles of API:

- The **`Deserialize…`** methods (`DeserializeResource`, `DeserializeElement`, `Deserialize<T>`) throw a `DeserializationFailedException` when there is anything to report.
- The **`TryDeserialize…`** methods return a `bool` and hand you the issues through an `out` parameter, without throwing.

The throwing style is the most common:

```csharp
var json = """{ "resourceType": "Patient", "id": "example-patient" }""";
var deserializer = FhirJsonDeserializer.DEFAULT;
try
{
    var patient = deserializer.DeserializeResource(json);
}
catch (DeserializationFailedException e)
{
    Console.WriteLine(e.Message);
}
```

`DeserializationFailedException` carries two useful properties:

- `Exceptions` — the full list of `CodedException`s detected. The deserializer continues past recoverable problems, so this is the *complete* set of issues, not just the first.
- `PartialResult` — a best-effort POCO containing as much data as could be deserialized. This lets you inspect or even keep working with the data despite the errors (bearing in mind some data may have ended up in overflow).

The `TryDeserialize…` form gives you the same information without an exception:

```csharp
if (!deserializer.TryDeserializeResource(reader, out var patient, out var issues))
{
    foreach (var issue in issues)
        Console.WriteLine($"{issue.ErrorCode}: {issue.Message}");
}
```

### The shape of an issue

Every issue is a `CodedException`, which always has:

- `ErrorCode` — a stable, permanent code (e.g. `JSON129`, `PVAL116`). These codes never change between versions, so they are safe to switch on in your own error handling.
- `Message` — a human-readable description.

The issues produced by the deserializers are in fact `ExtendedCodedException`s, which add location and severity information:

- `IssueSeverity` — `Fatal`, `Error` or `Warning` (see below).
- `IssueType` — the FHIR `OperationOutcome.IssueType`, handy when turning issues into an `OperationOutcome`.
- `InstancePath` — a simple FHIRPath to the offending element.
- `LineNumber` / `Position` — the location in the source data (populated when available).
- `BaseErrorMessage` / `Display` — the message without location, and a short display label.

### Severity and recoverability

The severity of an issue tells you whether data was lost:

| Severity | Meaning |
|----------|---------|
| `Fatal` | Data loss occurred and parsing cannot safely continue — for example a duplicate JSON property, or multiple resources inside a single contained element. |
| `Error` | A real FHIR conformance problem, but no data was lost — the data was captured, possibly in overflow. |
| `Warning` | A cosmetic issue that affects neither data nor structure — for example a disallowed `schemaLocation` attribute, or leading/trailing whitespace. |

This severity is what the deserialization *modes* use to decide which issues to ignore.

## Overflow and the `HasOverflow` guarantee

When a value cannot be placed in its typed property, it is stored in the object's overflow and the deserializer raises an issue whose code is in a known "overflow-causing" set (such as `UNKNOWN_ELEMENT` or `PROPERTY_TYPE_MISMATCH`). The practical consequence: **if none of those issues were raised, you can use the POCO's properties and primitive `Value` properties without them throwing, and `Base.HasOverflow` is `false`.** If they *were* raised, the data is still there — but you may need to read it from overflow.

The `NoOverflow` mode (below) exists precisely to make this guarantee explicit: it lets recoverable problems through but still fails on anything that would land in overflow.

## Deserialization modes

A *mode* is a preset that tells the deserializer which categories of issue to ignore. You select one through the `DeserializationMode` enum, or via the ready-made presets on the deserializer classes.

| Mode | Ignores | Model validation | Narrative (XHTML) |
|------|---------|------------------|-------------------|
| `Strict` | nothing — reports every issue | on | validated |
| `NoOverflow` | recoverable issues that do **not** cause overflow | on | not validated |
| `Recoverable` | all non-`Fatal` issues (no data loss) | on | not validated |
| `BackwardsCompatible` | unknown elements, attributes, codes, and choice types — the things a newer FHIR version might add | on | not validated |
| `SyntaxOnly` | nothing from the parser, but **model validation is switched off entirely** | off | not validated |
| `Ostrich` | everything, including `Fatal` data-loss issues | off | not validated |

`Strict` is the default for most operations: it reports everything. `Recoverable` is the usual choice for ingesting real-world data, `BackwardsCompatible` for reading data from other FHIR versions, and `Ostrich` for debugging, when you are certain the input is correct, or when correctness is simply not important to your use case — for example when you only need a few elements of a resource and do not care whether the rest is valid.

The deserializer classes expose these as static presets:

```csharp
var d1 = FhirJsonDeserializer.DEFAULT;             // strict, but XHTML narrative not validated
var d2 = FhirJsonDeserializer.STRICT;              // fully strict, including narrative
var d3 = FhirJsonDeserializer.NOOVERFLOW;
var d4 = FhirJsonDeserializer.RECOVERABLE;
var d5 = FhirJsonDeserializer.BACKWARDSCOMPATIBLE;
var d6 = FhirJsonDeserializer.SYNTAXONLY;
var d7 = FhirJsonDeserializer.OSTRICH;
```

To build a deserializer for a mode yourself, pass settings configured with `UsingMode`:

```csharp
var settings = new DeserializerSettings().UsingMode(DeserializationMode.Recoverable);
var deserializer = new FhirJsonDeserializer(settings);
```

### Fine-tuning: enforcing and ignoring individual codes

On top of a mode you can ignore or enforce individual error codes with `Ignoring` and `Enforcing`:

```csharp
// Recoverable, but additionally treat duplicate properties as acceptable
var settings = new DeserializerSettings()
    .UsingMode(DeserializationMode.Recoverable)
    .Ignoring([FhirJsonException.DUPLICATE_PROPERTY_CODE]);
```

These compose **left-to-right**, and a later `UsingMode` resets the filter. So the order of the calls matters:

```csharp
// 'Enforcing' after 'UsingMode' keeps the enforced code even in Ostrich mode
new DeserializerSettings()
    .UsingMode(DeserializationMode.Ostrich)
    .Enforcing([FhirJsonException.EXPECTED_PRIMITIVE_NOT_NULL_CODE]);

// 'UsingMode' after 'Enforcing' wipes out the enforcement again
new DeserializerSettings()
    .Enforcing([FhirJsonException.EXPECTED_PRIMITIVE_NOT_NULL_CODE])
    .UsingMode(DeserializationMode.Ostrich);
```

The codes themselves are defined as constants on `FhirJsonException`, `FhirXmlException` and `CodedValidationException`. Rather than reproduce a list here that could go stale, see {doc}`deserialization-behavior` for the categorised reference, and the source for the authoritative set.

(validation-during-parsing)=
## Validation during parsing

The model-validation issues (the `PVAL…` codes) are not produced by the parser but by a validator that the deserializer runs as it builds the POCO. By default this is `FhirAttributeValidator.Default`, which checks the structural rules expressed as attributes on the generated POCOs — cardinalities, coded values, string lengths, and so on.

Which mode you choose decides whether this validator runs at all: `SyntaxOnly` and `Ostrich` switch it off (so only — or no — parser issues are reported), while every other mode leaves it on and then filters its output by severity.

```{note}
This is *structural* validation only — it does not check FHIR profiles or invariants. Full profile validation is a separate step, described in the {ref}`validation chapter<poco-validation>`.
```
