(poco-validation)=
# Validating POCOs

The SDK can validate a POCO in memory against the structural rules of FHIR: cardinalities, the allowed types of choice elements, the format of primitive values, coded values against required bindings, and a handful of invariants. These are the rules that can be expressed in the generated C# model — so you get them without needing a terminology server or the full profile validator.

## Invoking validation

Add the `using` and call the `Validate()` extension on any POCO. It **returns** the validation problems as a collection — an empty result means the instance is valid. It does not throw:

```csharp
using Hl7.Fhir.Validation;

var errors = patient.Validate();
if (errors.Count > 0)
    // inspect the CodedValidationExceptions
```

By default this validates the whole object tree, uses the model for your project's FHIR version, and validates narrative XHTML.

```{note}
The deserializers run this same validation automatically while parsing (depending on the {doc}`mode </parsing/error-handling>`). A POCO obtained from a deserializer that did not report issues is therefore already validated — there is no need to call `Validate()` on it again.
```

## What is validated

- **Cardinality** — repeating elements have an allowed number of items, and mandatory elements are present.
- **Choice types** — a choice element (like `Observation.value[x]`) holds one of its allowed types.
- **Primitive formats** — primitive values are well-formed (e.g. a valid `date`, `instant`, or base64).
- **Coded values** — values bound to a required binding are valid members of the binding (these are generated as .NET enumerations).
- **Invariants** — a few checks not bound to a single property, such as contained resources not themselves containing resources.

For *how* these checks are implemented, and how to customize or replace the validator, see {doc}`poco-validation-internals`.

## What is not covered

The in-memory validation only covers what the C# model can express. Notably it does **not** cover:

* **Terminology** — it does not call a terminology service; it only checks required, explicit bindings.
* **Slices** — not used by the core resources, and not readily expressible in a .NET class model.
* **FhirPath invariants** — the additional `FhirPath` constraints in the FHIR definitions are not generated into the POCOs.

If you need these, use the {ref}`profile validator <profile-validation>`.

## Validation error codes

Every problem is reported as a `CodedValidationException`, which carries a human-readable message and a stable code (a `PVAL…` constant on `CodedValidationException`) that you can switch on programmatically. The codes do not change between SDK versions.

| Code | Constant | Checks that… |
|------|----------|--------------|
| `PVAL101` | `CHOICE_TYPE_NOT_ALLOWED` | a choice element's value is one of the allowed types. |
| `PVAL102` | `INCORRECT_CARDINALITY_MIN` | an element has at least its minimum number of items. |
| `PVAL103` | `INCORRECT_CARDINALITY_MAX` | an element has at most its maximum number of items. |
| `PVAL104` | `REPEATING_ELEMENT_CANNOT_CONTAIN_NULL` | a repeating element contains no null entries. |
| `PVAL105` | `MANDATORY_ELEMENT_MUST_BE_PRESENT` | a mandatory element is present. |
| `PVAL114` | `NARRATIVE_XML_IS_MALFORMED` | narrative is well-formed XML. |
| `PVAL115` | `NARRATIVE_XML_IS_INVALID` | narrative XML follows the FHIR narrative rules. |
| `PVAL116` | `INVALID_CODED_VALUE` | a code is valid for its value set. |
| `PVAL118` | `CONTAINED_RESOURCES_CANNOT_BE_NESTED` | contained resources do not themselves contain resources. |
| `PVAL119` | `INVALID_STRING_LENGTH` | a string is neither empty nor over the maximum length. |
| `PVAL120` | `INVALID_BASE64_VALUE` | a value is parseable as base64. |
| `PVAL123` | `INCORRECT_LITERAL_VALUE_TYPE` | a literal is the right kind for its primitive type. |
| `PVAL124` | `LITERAL_INVALID` | a literal is a valid value for its primitive type. |
| `PVAL125` | `POSITIVE_INT_MUST_BE_POSITIVE` | a `positiveInt` is positive. |
| `PVAL126` | `UNSIGNED_INT_MUST_NOT_BE_NEGATIVE` | an `unsignedInt` is not negative. |
| `PVAL127` | `PROPERTY_TYPE_MISMATCH` | a value's type matches its element. |
| `PVAL128` | `UNKNOWN_ELEMENT` | an element is defined on the type. |
| `PVAL129` | `ELEMENT_CANNOT_BE_EMPTY` | an element actually carries a value. |
| `PVAL130` | `UNKNOWN_RESOURCE_TYPE` | a resource's type is recognized. |

Rather than depend on this list staying complete, the authoritative set is [in the source](https://github.com/FirelyTeam/firely-net-sdk/blob/develop/src/Hl7.Fhir.Base/Validation/CodedValidationException.cs).
