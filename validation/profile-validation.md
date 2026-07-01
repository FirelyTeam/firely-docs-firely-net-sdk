(profile-validation)=
# Validating data against profiles

The profile validator is distributed as a separate package, and can be found on NuGet, by searching for `Firely.Fhir.Validation.<spec version>`. The main class in this package is the `Validator`. To construct a new instance, we can supply the constructor with several dependencies:

* Required: A reference to a resource resolver (see {ref}`fhir-package-source`), specifically an `IAsyncResourceResolver`. This service allows the validator to retrieve the profiles that are referenced by the profile being validated.
* Required: A reference to a terminology service (see {ref}`terminology-service`), specifically a `ICodeValidationTerminologyService`. This service is needed to validate the codes used in the instance against the code systems and value sets specified in the profile.
* Optional, but recommended: pass in an `IExternalReferenceResolver`. This service allows the validator to retrieve the resources that are referenced by the instance being validated. If this service is not supplied, the validator will simply not try to validate the resources that these references point to.
* Optional: a set of additional settings, as an instance of a `ValidationSettings`.

```{note}
The validator translates the StructureDefinitions to an optimized, internal representation, which are cached in the validator. It is therefore recommended to reuse the same instance of the validator for multiple validations.
```

```csharp
var packageServer = "https://packages.simplifier.net";
var fhirRelease = FhirRelease.STU3;
var packageResolver = FhirPackageSource.CreateCorePackageSource(ModelInfo.ModelInspector, fhirRelease, packageServer);
var resourceResolver = new CachedResolver(packageResolver);
var terminologyService = new LocalTerminologyService(resourceResolver);

var validator = new Validator(resourceResolver, terminologyService);
var testOrganization = new Organization { };

var profile = Canonical.ForCoreType("Organization").ToString();
var result = validator.Validate(testOrganization, profile);
```

The above code initializes a package resolver, which is configured to retrieve the basic core FHIR STU3 packages. It also creates a terminology service, which is configured to use the same resource resolver to retrieve the code systems and value sets that are referenced by the profiles. Finally, it creates a validator, and runs a validation against a new instance of the Organization resource, using the standard Organization profile.

In practice, you would configure additional resolvers (using a `MultiResolver`), which could combine the core packages with profiles found on a shared drive storage, a database etcetera.

## Validation results

The validator returns a FHIR `OperationOutcome` resource. It has an additional `Success` property that can be checked to see if there are any important issues. The `Fatals`, `Errors` and `Warnings` properties contain the number of issues that were found during validation for each category. Because the result is an `OperationOutcome`, its `Issue` entries carry a coding you can test against, which makes it easy to check for a specific kind of error:

```csharp
var result = validator.Validate(p);
result.Issue.SelectMany(i => i.Details.Coding).Should().Contain(c => c.Code == Issue.CONTENT_ELEMENT_CHOICE_INVALID_INSTANCE_TYPE.Code.ToString());
result.Success.Should().BeFalse();
```

## External references

FHIR instances can contain references to other resources, which can be used to validate the instance. For example, a Patient resource can contain a reference to a Practitioner. The validator can be configured with an `IExternalReferenceResolver` to retrieve these resources. If this service is not supplied, the validator will simply not try to validate the resources that these references point to.

The interface looks like this:

```csharp
/// <summary>
/// A service that can resolve external references to other resources.
/// </summary>
public interface IExternalReferenceResolver
{
    /// <summary>
    /// Resolves the reference to a resource. The returned object must be either a <see cref="Resource"/> or <see cref="ElementNode"/>
    /// </summary>
    /// <returns>The resource or element node, or null if the reference could not be resolved.</returns>
    Task<object?> ResolveAsync(string reference);
}
```

When implementing this interface, you can return either a `Resource` or an `ElementNode`, depending on whether you are working with POCO's or `ITypedElement`-based models. Return `null` if the reference cannot be resolved.

## Selecting profiles to validate against

In the first `Validate()` example above, we passed an explicit profile url to validate against. This is not always necessary. If you leave out the profile url, the validator will try to find a profile url in the `Meta` of the instance. If it finds one, it will validate against that profile. If it does not find one, it will validate against the "default" core profile for the resource type. You would normally only pass in an explicit profile url if you want to validate against a specific profile, e.g. one for US Core or other national profiles.

The behaviour of following the profiles in Meta can be changed by setting the `SelectValidationProfiles` property.

## Other configuration operations

You can pass in an instance of the `ValidationSettings` class to the constructor of the `Validator`. This class contains several properties that can be used to configure the behaviour of the validator. The most important ones are:

| Property | Use |
|----------|-----|
| ConstraintBestPractices | Determines how to deal with failures of FhirPath constraints marked as "best practice". Default is `Warning`. |
| SelectValidationProfiles | Determines which profiles to validate the instance against. If not set, the profiles in the instance's `Meta` are used. |
| FollowExtensionUrl | Determines what to do when an extension is encountered. If not set, a validation of an Extension will warn if the extension cannot be resolved, or return an error when it cannot be resolved and is a modifier extension. |
| TypeNameMapper | A function that maps a type name found in `TypeRefComponent.Code` to a resolvable canonical. If not set, it will prefix the type with the standard `http://hl7.org/fhir/StructureDefinition` prefix. |
| IncludeFilters / ExcludeFilters | Predicates that decide which assertions are run, e.g. to skip particular FhirPath constraints. By default an exclude filter suppresses the `dom-6` invariant. |

## Full Example

We have created a full example that shows how to use the validator using a terminology service and the FHIR core package resolvers. See [this GitHub repo](https://github.com/FirelyTeam/Firely.Fhir.ValidationDemo) for more information.

## Beyond the Validator

The `Validator` is a high-level wrapper over a lower-level engine that compiles StructureDefinitions into schemas and runs them against instances. For most callers it is the right entry point. If you need finer control — tuning schema caching, adding custom compilation rules, or working with the raw validation result — see {doc}`profile-validation-internals`.
