(deserialization-behavior)=
# Deserialization behavior reference

This page is a reference for the exact behavior of the SDK 6 deserializers when they encounter an irregularity. It complements {doc}`error-handling`, which explains the concepts; here we list the individual issue codes, their severity, and — importantly — what the deserializer *does* with the data in each case.

All codes are stable: the `ErrorCode` of an issue never changes between SDK versions, so it is safe to refer to these constants in your own error handling. The constants live on `FhirJsonException`, `FhirXmlException` and `CodedValidationException`; the lists below mirror those classes.

## JSON syntax issues

These are raised by the JSON parser as `FhirJsonException`. Unless marked `Fatal`, all data is preserved.

| Code | Constant | Severity | Behavior |
|------|----------|----------|----------|
| `JSON102` | `RESOURCETYPE_SHOULD_BE_STRING` | Error | The resource is stored as a `DynamicResource`. |
| `JSON103` | `NO_RESOURCETYPE_PROPERTY` | Error | No `resourceType` found; reported but parsing continues. |
| `JSON109` | `EXPECTED_PRIMITIVE_NOT_NULL` | Error | A JSON `null` where a value was expected; the null carries no data, so it is dropped. |
| `JSON114` | `CHOICE_ELEMENT_MUST_HAVE_SUFFIX` | Error | A choice element with no type suffix; the value is stored using a `Dynamic` type. |
| `JSON120` | `OBJECTS_CANNOT_BE_EMPTY` | Error | An empty object is ignored; no data to capture. |
| `JSON121` | `ARRAYS_CANNOT_BE_EMPTY` | Error | An empty array is ignored; no data to capture. |
| `JSON125` | `PRIMITIVE_ARRAYS_ONLY_NULL` | Error | Array of only nulls; the nulls are left in place, no change in data. |
| `JSON129` | `DUPLICATE_PROPERTY` | **Fatal** | A property appears twice; this loses data, so parsing of the object cannot safely continue. |
| `JSON130` | `NESTED_ARRAY` | Error | Nested arrays are flattened, so no data is lost. |
| `JSON131` | `UNEXPECTED_PRIMITIVE_VALUE_FOR_NON_PRIMITIVE` | Error | A primitive was found where an object was expected. |
| `JSON132` | `UNEXPECTED_OBJECT_VALUE_FOR_PRIMITIVE` | Error | An object was found where a primitive was expected. |
| `JSON133` | `USE_OF_UNDERSCORE_WITH_NON_PRIMITIVE` | Error | The underscore form was used on a non-primitive; the underscore is ignored and parsing continues. |
| `JSON134` | `UNDERSCORE_SHOULD_BE_OBJECT` | Error | An underscore property that was not an object; the primitive value is added as the element's `value`. |

## XML syntax issues

These are raised by the XML parser as `FhirXmlException`. XML has concerns JSON does not — namespaces, element ordering, and the attribute/element distinction — so it has a few more codes.

| Code | Constant | Severity | Behavior |
|------|----------|----------|----------|
| `XML101` | `EMPTY_ELEMENT_NAMESPACE` | Error | Element has no namespace; parsing continues as if it were the FHIR namespace. |
| `XML105` | `CHOICE_ELEMENT_MUST_HAVE_SUFFIX` | Error | Choice element without a type suffix; stored using a `Dynamic` type. |
| `XML107` | `INCORRECT_XHTML_NAMESPACE` | Error | Narrative uses the wrong namespace; reported but captured. |
| `XML109` | `ELEMENT_OUT_OF_ORDER` | Error | Elements in the wrong order; the data is still parsed safely. |
| `XML110` | `MULTIPLE_ELEMENTS_IN_RESOURCE_CONTAINER` | **Fatal** | More than one resource inside a container element; data loss. |
| `XML111` | `NO_ATTRIBUTES_ALLOWED_ON_RESOURCE_CONTAINER` | **Fatal** | Unexpected attribute on a resource container; data loss. |
| `XML112` | `INCORRECT_ELEMENT_NAMESPACE` | Error | Element uses a disallowed namespace. |
| `XML113` | `DISALLOWED_NODE_TYPE` | **Fatal** | An XML node type that is not allowed at this point. |
| `XML114` | `INCORRECT_ATTRIBUTE_NAMESPACE` | Error | Attribute uses a disallowed namespace. |
| `XML116` | `ELEMENT_NOT_IN_SEQUENCE` | Error | A repeating element appears more than once but not contiguously. |
| `XML117` | `SCHEMALOCATION_DISALLOWED` | Warning | A `schemaLocation` attribute; ignored, carries no data. |
| `XML122` | `EMPTY_RESOURCE_CONTAINER` | Error | An empty resource container; reported, no data to capture. |
| `XML124` | `ELEMENT_SHOULD_HAVE_BEEN_AN_ATTRIBUTE` | Error | Encoded as an element but should be an attribute; the content is captured anyway. |
| `XML125` | `ATTRIBUTE_SHOULD_HAVE_BEEN_AN_ELEMENT` | Error | Encoded as an attribute but should be an element; the content is captured anyway. |
| `XML126` | `STRING_SHOULD_NOT_HAVE_LEADING_OR_TRAILING_WHITESPACE` | Warning | Whitespace around a string value; trimmed during parsing. |

## Differences between the XML and JSON deserializers

The two formats share the same model and modes, but a few behaviors are format-specific:

- **Namespaces and ordering** are XML-only concerns (`XML101`, `XML107`, `XML109`, `XML112`, `XML114`).
- The **attribute vs. element** distinction (`XML124`, `XML125`) has no JSON equivalent.
- JSON's **underscore form** for primitive extensions (`JSON133`, `JSON134`) and its **duplicate-property** problem (`JSON129`) have no XML equivalent.

Otherwise both produce the same model-validation issues and obey the same modes.

<!-- TODO(pval-relocate): The PVAL model-validation codes below belong to the validation chapter.
     When that chapter is (re)written, move the authoritative table there and leave only a
     cross-reference here. See memory note pval-codes-relocate-to-validation. -->

## Model-validation issues

```{note}
These issues come from the validator that runs during parsing (see {ref}`validation-during-parsing`), not from the parser itself. They are covered in depth in the {ref}`validation chapter<poco-validation>`; the table here is a quick reference for how they surface during deserialization.
```

These are raised as `CodedValidationException` (codes `PVAL…`). The ones marked **overflow** in the last column are the issues that indicate data did *not* fit a typed property and was placed in overflow — in other words, if none of these were raised, the POCO can be used without checking `Base.Overflow`.

| Code | Constant | Causes overflow |
|------|----------|-----------------|
| `PVAL101` | `CHOICE_TYPE_NOT_ALLOWED` | |
| `PVAL102` | `INCORRECT_CARDINALITY_MIN` | |
| `PVAL103` | `INCORRECT_CARDINALITY_MAX` | |
| `PVAL104` | `REPEATING_ELEMENT_CANNOT_CONTAIN_NULL` | |
| `PVAL105` | `MANDATORY_ELEMENT_MUST_BE_PRESENT` | |
| `PVAL114` | `NARRATIVE_XML_IS_MALFORMED` | |
| `PVAL115` | `NARRATIVE_XML_IS_INVALID` | |
| `PVAL116` | `INVALID_CODED_VALUE` | yes |
| `PVAL118` | `CONTAINED_RESOURCES_CANNOT_BE_NESTED` | |
| `PVAL119` | `INVALID_STRING_LENGTH` | yes |
| `PVAL120` | `INVALID_BASE64_VALUE` | yes |
| `PVAL123` | `INCORRECT_LITERAL_VALUE_TYPE` | yes |
| `PVAL124` | `LITERAL_INVALID` | yes |
| `PVAL125` | `POSITIVE_INT_MUST_BE_POSITIVE` | |
| `PVAL126` | `UNSIGNED_INT_MUST_NOT_BE_NEGATIVE` | |
| `PVAL127` | `PROPERTY_TYPE_MISMATCH` | yes |
| `PVAL128` | `UNKNOWN_ELEMENT` | yes |
| `PVAL129` | `ELEMENT_CANNOT_BE_EMPTY` | |
| `PVAL130` | `UNKNOWN_RESOURCE_TYPE` | |

Of these, the ones allowed through in `BackwardsCompatible` mode — because a newer FHIR version could legitimately introduce them — are `INVALID_CODED_VALUE`, `UNKNOWN_ELEMENT`, `CHOICE_TYPE_NOT_ALLOWED` and `UNKNOWN_RESOURCE_TYPE`.
