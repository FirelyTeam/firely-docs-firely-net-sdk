# Parsing and serialization

The Firely .NET SDK reads and writes FHIR data in both JSON and XML. Throughout this section the examples use JSON — by far the most common format. XML works the same way through the equivalent classes, and we point out the few places where it differs.

Almost all work with FHIR data goes through the POCO classes (see {ref}`FHIR-model`): .NET classes such as `Patient` that mirror the FHIR resources. The SDK turns serialized FHIR into these POCOs and back again:

- {doc}`/parsing/deserialization` — reading JSON or XML into POCOs.
- {doc}`/parsing/error-handling` — how the deserializer reports problems, and the modes that control how strict it is.

```{note}
The SDK 6 deserializers are built to capture *all* incoming data, even when it does not perfectly fit the POCO — unknown elements and mismatched values are preserved in the element's *overflow* (see {doc}`/model/dynamic-features`) rather than discarded. As a result, the POCO approach handles invalid or cross-version data that older SDKs could not.
```

For low-level, version-independent access to FHIR data as a generic tree, there is the older {doc}`ElementModel </parsing/elementmodel>` API, kept for backward compatibility.
