(deserialization-behavior)=
# Deserialization behavior reference

This page is a reference for the exact behavior of the SDK 6 deserializers when they encounter an irregularity. It complements {doc}`error-handling`, which explains the concepts; here we list the individual issue codes, their severity, and — importantly — what the deserializer *does* with the data in each case.

All codes are stable: the `ErrorCode` of an issue never changes between SDK versions, so it is safe to refer to these constants in your own error handling. The constants live on `FhirJsonException`, `FhirXmlException` and `CodedValidationException`; the lists below mirror those classes.

## JSON syntax issues

These are raised by the JSON parser as `FhirJsonException`. Unless marked `Fatal`, all data is preserved.

| Code | Constant | Severity | What happens to the data |
|------|----------|----------|--------------------------|
| `JSON102` | `RESOURCETYPE_SHOULD_BE_STRING` | Error | The non-string value is used as the type name and the resource is stored as a `DynamicResource`. |
| `JSON103` | `NO_RESOURCETYPE_PROPERTY` | Error | With no type to resolve, the resource is stored as an (unnamed) `DynamicResource`. |
| `JSON109` | `EXPECTED_PRIMITIVE_NOT_NULL` | Error | The `null` is skipped. It carries no data, so nothing is lost. |
| `JSON114` | `CHOICE_ELEMENT_MUST_HAVE_SUFFIX` | Error | The type is guessed and the value is stored using a `Dynamic` type. |
| `JSON120` | `OBJECTS_CANNOT_BE_EMPTY` | Error | The empty object holds no data; it is skipped. |
| `JSON121` | `ARRAYS_CANNOT_BE_EMPTY` | Error | The empty array holds no data; it is skipped. |
| `JSON125` | `PRIMITIVE_ARRAYS_ONLY_NULL` | Error | The nulls are left in place; there was no data to capture. |
| `JSON129` | `DUPLICATE_PROPERTY` | **Fatal** | The duplicates are merged into a single element; a conflicting primitive value keeps the first occurrence, so the later data is dropped. |
| `JSON130` | `NESTED_ARRAY` | Error | The nested array is flattened into its parent, so no data is lost. |
| `JSON131` | `UNEXPECTED_PRIMITIVE_VALUE_FOR_NON_PRIMITIVE` | Error | The primitive cannot fit the complex element, so it is preserved in that element's **overflow** under the key `value`. |
| `JSON132` | `UNEXPECTED_OBJECT_VALUE_FOR_PRIMITIVE` | Error | The object is deserialized into the primitive: `id` and `extension` are set normally, and every other member — including a stray `value` — is preserved in the primitive's **overflow**. |
| `JSON133` | `USE_OF_UNDERSCORE_WITH_NON_PRIMITIVE` | Error | The underscore form is only valid for primitives; here it is ignored and the complex value is parsed normally. |
| `JSON134` | `UNDERSCORE_SHOULD_BE_OBJECT` | Error | The `_`-prefixed companion should be an object; the stray primitive is used as the element's value. If the non-underscore property already set the value, the stray primitive is silently dropped (this remains a non-fatal error). |

```{note}
When a value cannot be placed in its typed property — as with `JSON131` and `JSON132` — the generated POCO keeps the value in the element's *overflow* and leaves the typed property empty. The data is therefore not lost, but it must be read through the overflow (see {doc}`/model/dynamic-features`), `Base.HasOverflow` becomes `true`, and the model validator additionally reports this as an overflow-causing `PROPERTY_TYPE_MISMATCH` (`PVAL127`).
```

```{note}
The `JSON133` and `JSON134` codes concern FHIR's *underscore form* for primitives. In FHIR JSON a primitive element such as `name` may be accompanied by a `_name` companion that carries its `id` and `extension`; the two are merged into a single element. The companion is only valid on a primitive (otherwise `JSON133`) and must be a JSON object (otherwise `JSON134`).
```

## XML syntax issues

These are raised by the XML parser as `FhirXmlException`. XML has concerns JSON does not — namespaces, element ordering, and the attribute/element distinction — so it has a few more codes.

| Code | Constant | Severity | What happens to the data |
|------|----------|----------|--------------------------|
| `XML101` | `EMPTY_ELEMENT_NAMESPACE` | Error | The element is parsed as though it were in the FHIR namespace. |
| `XML105` | `CHOICE_ELEMENT_MUST_HAVE_SUFFIX` | Error | The type is guessed and the value is stored using a `Dynamic` type. |
| `XML107` | `INCORRECT_XHTML_NAMESPACE` | Error | The narrative XHTML is still read and captured. |
| `XML109` | `ELEMENT_OUT_OF_ORDER` | Error | The element is parsed normally; order does not affect the result. |
| `XML110` | `MULTIPLE_ELEMENTS_IN_RESOURCE_CONTAINER` | **Fatal** | Only one resource fits in a container, so the extra resources are lost. |
| `XML111` | `NO_ATTRIBUTES_ALLOWED_ON_RESOURCE_CONTAINER` | **Fatal** | The attribute cannot be represented on a container, so its data is lost. |
| `XML112` | `INCORRECT_ELEMENT_NAMESPACE` | Error | The element is parsed anyway, ignoring the namespace. |
| `XML113` | `DISALLOWED_NODE_TYPE` | **Fatal** | The node cannot be represented in the model, so its content is lost. |
| `XML114` | `INCORRECT_ATTRIBUTE_NAMESPACE` | Error | The attribute value is parsed anyway, ignoring the namespace. |
| `XML116` | `ELEMENT_NOT_IN_SEQUENCE` | Error | The value is still added to the (repeating) element. |
| `XML117` | `SCHEMALOCATION_DISALLOWED` | Warning | The attribute is ignored; it carries no data. |
| `XML122` | `EMPTY_RESOURCE_CONTAINER` | Error | The container holds no resource; there is nothing to capture. |
| `XML124` | `ELEMENT_SHOULD_HAVE_BEEN_AN_ATTRIBUTE` | Error | The element's content is captured into the value anyway. |
| `XML125` | `ATTRIBUTE_SHOULD_HAVE_BEEN_AN_ELEMENT` | Error | The attribute's content is captured into the element anyway. |
| `XML126` | `STRING_SHOULD_NOT_HAVE_LEADING_OR_TRAILING_WHITESPACE` | Warning | The whitespace is trimmed and the trimmed value is kept. |

## Differences between the XML and JSON deserializers

The two formats share the same model and modes, but a few behaviors are format-specific:

- **Namespaces and ordering** are XML-only concerns (`XML101`, `XML107`, `XML109`, `XML112`, `XML114`).
- The **attribute vs. element** distinction (`XML124`, `XML125`) has no JSON equivalent.
- JSON's **underscore form** for primitive extensions (`JSON133`, `JSON134`) and its **duplicate-property** problem (`JSON129`) have no XML equivalent.

Otherwise both produce the same model-validation issues and obey the same modes.

## Model-validation issues

Beyond the syntax issues above, the validator that runs during parsing (see {ref}`validation-during-parsing`) raises model-validation issues as `CodedValidationException` (codes `PVAL…`). The full list of these codes lives in the {ref}`validation chapter <poco-validation>`; this section only covers how they surface during deserialization.

The following codes indicate that data did **not** fit a typed property and was captured in overflow — if none of these were raised, the POCO can be used without checking `Base.Overflow`:

- `INVALID_CODED_VALUE` (`PVAL116`)
- `INVALID_STRING_LENGTH` (`PVAL119`)
- `INVALID_BASE64_VALUE` (`PVAL120`)
- `INCORRECT_LITERAL_VALUE_TYPE` (`PVAL123`)
- `LITERAL_INVALID` (`PVAL124`)
- `PROPERTY_TYPE_MISMATCH` (`PVAL127`)
- `UNKNOWN_ELEMENT` (`PVAL128`)

The issues allowed through in `BackwardsCompatible` mode — because a newer FHIR version could legitimately introduce them — are `INVALID_CODED_VALUE`, `UNKNOWN_ELEMENT`, `CHOICE_TYPE_NOT_ALLOWED` and `UNKNOWN_RESOURCE_TYPE`.
