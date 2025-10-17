# Equality and Cloning

The SDK includes features for comparing FHIR model instances for equality and creating deep copies (clones) of these instances.

## Equality

FHIR model classes do not directly implement equality comparison methods such as `IEquatable`, `Equals`, or the `==` operator. Instead, the SDK provides utility methods on each class to compare FHIR model instances based on their content.

### `IsExactly` and `Matches`

The primary method for comparing two FHIR model instances is the `IsExactly` method. This performs a deep comparison of all properties and elements, ensuring that the instances are identical in both data and structure.

`IsExactly` treats `null` values and empty lists as equivalent. For instance, a property that is `null` in one instance and an empty list in another will be considered equal.

The `Matches` method offers a more lenient comparison against a "pattern," allowing for some differences between the instance and the pattern:

- If the pattern has a property set to `null`, the corresponding property in the instance is ignored during comparison.
- If the pattern has an empty list property, the corresponding list in the instance is ignored during comparison.
- If the pattern has a list property with elements, the instance's corresponding list must contain at least those elements but may include additional elements.

```{note}
For equality comparison, both `IsExactly` and `Matches` ignore the instance's [annotations](other-features).
```

### Customizing Equality Comparison

The FHIR specification does not define equivalence between two resources. The `IsExactly` and `Matches` methods align with the FHIR specification's definitions of equality for [`ElementDefinition.fixed[x]`](https://hl7.org/fhir/elementdefinition-definitions.html#ElementDefinition.fixed_x_) and [`ElementDefinition.pattern[x]`](https://hl7.org/fhir/elementdefinition-definitions.html#ElementDefinition.pattern_x_).

However, you may need to customize comparison behavior for specific properties or elements. For example, you might want to ignore certain metadata properties or treat specific elements differently. 



The SDK uses the .NET `IEqualityComparer<T>` interface to enable custom equality comparison logic. To customize equality for a specific type, implement your own comparer by implementing both `IEqualityComparer<Base>` and `IEqualityComparer<IReadOnlyCollection<Base>>`. These implementations should define custom `Equals` and `GetHashCode()` methods. Typically, you will call the virtual `Base.CompareChildren()` method to perform a recursive ("deep") comparison of child elements, as shown below:

```csharp
public class MyCustomEqualityComparer : IEqualityComparer<Base>, IEqualityComparer<IReadOnlyCollection<Base>>
{
    public bool Equals(Base? x, Base? y)
    {
        if (ReferenceEquals(x, y)) return true;
        if (x is null || y is null) return false;
        if (x.GetType() != y.GetType()) return false;

        return x.CompareChildren(y, this);
    }

    public int GetHashCode(Base obj) => obj.GetHashCode();

    public bool Equals(IReadOnlyCollection<Base>? x, IReadOnlyCollection<Base>? y)
    {
        // Implement collection equality comparison
        throw new NotImplementedException();
    }

    public int GetHashCode(IReadOnlyCollection<Base> obj)
    {
        // Implement a suitable hash code calculation for collections
        throw new NotImplementedException();
    }
}
```

```{tip}
The `ExactMatchEqualityComparer` implements the logic for `IsExactly`, and the `PatternMatchEqualityComparer` implements the logic for `Matches`. You can use these comparers directly for equality comparisons outside the model classes.
```

## Cloning

The SDK provides the `DeepCopy()` method on all model classes to create deep copies (clones) of FHIR model instances. This method creates a new instance of the same type and recursively copies all properties and elements, ensuring the cloned instance is entirely independent of the original.

Additionally, the `CopyTo()` method allows you to copy the content of one instance to another existing instance of the same type. This is useful when reusing an existing instance instead of creating a new one. Note that the target instance will be overwritten with the source instance's content. Specifically, lists in the target instance will be replaced with copies of the source instance's lists.

```{note}
The `DeepCopy()` and `CopyTo()` methods also copy annotations attached to the original instance.
``` 