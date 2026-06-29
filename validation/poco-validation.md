(poco-validation)=
# Validating POCOs

The SDK can validate a POCO in memory against the structural rules of FHIR: cardinalities, the allowed types of choice elements, the format of primitive values, coded values against required bindings, and a handful of invariants. This catches the mistakes that a `StructureDefinition` for the core resources and datatypes expresses, without needing a terminology server or the full profile validator.

```{note}
Despite what older documentation said, this validation does **not** use .NET's `System.ComponentModel.DataAnnotations` framework. The SDK has its own validator, `FhirAttributeValidator` (an `IPocoValidator`), which is modelled on that pattern but works off the cached `ClassMapping`/`PropertyMapping` metadata rather than reflection. The POCOs do not implement `IValidatableObject`.
```

## Invoking validation

Add the `using` and call the `Validate()` extension on any POCO. It **returns** the validation problems as a collection — an empty result means the instance is valid. It does not throw:

```csharp
using Hl7.Fhir.Validation;

var errors = patient.Validate();
if (errors.Count > 0)
    // inspect the CodedValidationExceptions
```

By default this validates the whole object tree, uses the model for your project's FHIR version, and validates narrative XHTML. The overload taking a `ModelInspector` additionally lets you opt out of recursion (`validateRecursively`) and supply a custom `IPocoValidator`.

```{note}
The deserializers run this same validation automatically while parsing (depending on the {doc}`mode </parsing/error-handling>`). A POCO obtained from a deserializer that did not report issues is therefore already validated — there is no need to call `Validate()` on it again.
```

## What is validated

Two pieces of metadata participate as attributes (both derive from the SDK's `ValidatingFhirModelAttribute`, not from .NET's `ValidationAttribute`):

| Attribute | Validation performed |
|-----------|----------------------|
| `AllowedTypesAttribute` | On a "choice" element (like `Observation.value[x]`), verifies the instance's type is one of the allowed choices. |
| `CardinalityAttribute` | Verifies the number of items in a repeating element conforms to the element's min/max cardinality. |

Everything else is validated by the types themselves and the validator's structural checks:

- **Primitive value formats** — each primitive type validates its own value (e.g. a malformed `date`, `instant`, or invalid base64), so there are no separate per-format attributes.
- **Coded values** — values bound to a required binding are generated as .NET enumerations and validated against them.
- **Property types** — the value assigned to an element must be type-compatible with it.
- **Unknown content** — elements not defined on the type, and unknown resource types.
- **Object invariants** — checks not bound to a single property, such as contained resources not themselves containing resources, via `Base.ValidateInvariants()`.

## What is not covered

The in-memory validation does not cover the constraints that need more than the core metadata. Notably:

* **Terminology** — it does not call a terminology service; it only checks required, explicit bindings (which are generated as enumerations).
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
