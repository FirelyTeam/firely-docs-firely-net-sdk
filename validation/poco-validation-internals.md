(poco-validation-internals)=
# How POCO validation works

This page explains the mechanism behind {doc}`poco-validation` — useful if you want to understand exactly what runs, or to customize or replace the validator. For just running validation, the previous page is all you need.

## The pipeline

The `Validate()` extension builds a `PocoValidationContext` (carrying the path, optional line info, and narrative-validation setting) and walks the object tree, delegating the actual checks to an `IPocoValidator`. The deserializers use the same interface, which is why deserialization and explicit validation produce consistent results (see {ref}`validation-during-parsing`).

## IPocoValidator

The validator is an implementation of `IPocoValidator`, which has two methods:

```csharp
public interface IPocoValidator
{
    IReadOnlyCollection<CodedValidationException> ValidateProperty(
        string name, object? propertyValue, PropertyMapping? propertyMapping, PocoValidationContext context);

    IReadOnlyCollection<CodedValidationException> ValidateObject(
        Base instance, ClassMapping classMapping, PocoValidationContext context);
}
```

`ValidateProperty` runs on each element value as it is set; `ValidateObject` runs on the object as a whole (including primitive value checks and detecting missing mandatory elements). Both return their findings rather than throwing.

## FhirAttributeValidator

`FhirAttributeValidator` is the default implementation. It is modelled on .NET's `Validator.ValidateObject()`, but **does not use `System.ComponentModel.DataAnnotations`** — instead of reflection it works off the cached `ClassMapping`/`PropertyMapping` metadata, which is much faster. Its work falls into three parts:

- **Metadata checks** — unknown elements (`UNKNOWN_ELEMENT`), unknown resource types (`UNKNOWN_RESOURCE_TYPE`), and property type compatibility (`PROPERTY_TYPE_MISMATCH`).
- **Attribute validation** — running the validation attributes found on the property and the class (see below).
- **Native validation** — calling the instance's own `ValidateInvariants()` (and, for primitives, value validation).

## Validation attributes

Only two attributes take part in validation. Both derive from the SDK's own `ValidatingFhirModelAttribute` — **not** from .NET's `ValidationAttribute` — which declares a single method:

```csharp
public abstract IReadOnlyCollection<CodedValidationException> Validate(object? value, PocoValidationContext context);
```

| Attribute | Validation performed |
|-----------|----------------------|
| `AllowedTypesAttribute` | On a choice element, the instance's type is one of the allowed choices. |
| `CardinalityAttribute` | The number of items conforms to the element's min/max cardinality. |

You can make a custom attribute participate by deriving from `ValidatingFhirModelAttribute` and implementing `Validate()`.

## Native validation on the POCOs

Most validation is not attribute-based at all — it lives in virtual methods on the model types themselves, which both the validator and the deserializer invoke:

- `Base.ValidateInvariants(PocoValidationContext)` — object-level invariants that cannot be expressed per property (for example, that contained resources do not themselves contain resources). Override it on a type to add cross-property checks.
- `PrimitiveType.ValidateObjectValue(PocoValidationContext?)` — validates a primitive's raw value (its format/regex). This is where the per-type checks for `date`, `base64Binary`, and the like live — replacing the per-format attributes that older SDKs used.

## Plugging in a custom validator

Both the `Validate()` extension and the deserializers accept an `IPocoValidator`. Pass your own implementation to validate (or deserialize) with custom logic:

```csharp
var errors = patient.Validate(validator: new MyValidator());
```

During deserialization, set it via `DeserializerSettings.Validator` (see {doc}`/parsing/error-handling`). Setting that property to `null` disables model validation entirely — which is exactly what the `SyntaxOnly` and `Ostrich` modes do.
