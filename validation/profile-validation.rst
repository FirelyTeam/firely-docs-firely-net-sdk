.. _profile-validation:

================================
Validating data against profiles
================================

The profile validator is distributed as a separate package, and can be found on NuGet, by searching for ``Firely.Fhir.Validation.<spec version>``. The main class in this package is the `Validator`. To construct a new instance, we can supply the constructor with several dependencies:

* Required: A reference to a terminology service (see :ref:`terminology-service`), specifically a ``ICodeValidationTerminologyService``. This service is needed to validate the codes used in the instance against the code systems and value sets specified in the profile.
* Required: A reference to a resource resolver (see :ref:`package-source`), specifically an ``IAsyncResourceResolver``. This service allows the validator to retrieve the profiles that are referenced by the profile being validated.
* Optional, but recommended: pass in an ``IExternalReferenceResolver``. This service allows the validator to retrieve the resources that are referenced by the instance being validated. If this service is not supplied, the validator will simply not try to validate the resources that these references point to.
* Optional: pass in a configured ``FhirPathCompiler``. This allows you to configure the FhirPath compiler that is used to evaluate the FhirPath expressions in the profile. If not supplied, the default compiler will be used.

.. note::

   The validator translates the StructureDefinitions to an optimized, internal representation, which are cached in the validator. It is therefore recommended to reuse the same instance of the validator for multiple validations.


.. code-block:: csharp

    var packageServer = "https://packages.simplifier.net";
    var fhirRelease = FhirRelease.STU3;
    var packageResolver = FhirPackageSource.CreateCorePackageSource(ModelInfo.ModelInspector, fhirRelease, packageServerUrl);
    var resourceResolver = new CachedResolver(packageResolver);
    var terminologyService = new LocalTerminologyService(resourceResolver);

    var validator = new Validator(resourceResolver, terminologyService);
    var testOrganization = new Organization { };

    var result = validator.Validate(testOrganization, "http://hl7.org/fhir/StructureDefinition/Organization");

The above code initializes a package resolver, which is configured to retrieve the basic core FHIR STU3 packages. It also creates a terminology service, which is configured to use the same resource resolver to retrieve the code systems and value sets that are referenced by the profiles. Finally, it creates a validator, and runs a validation against a new instance of the Organization resource, using the standard Organization profile. 

In practice, you would configure additional resolvers (using a ``MultiResolver``), which could combine the core packages with profiles found on a shared drive storage, a database etcetera.

Validation results
------------------

The validator returns a ``ResultReport`` object, of whichs the ``Result`` property is the most important. This contains either ``Success``, ``Failure`` or ``Undecided``. The ``Success`` and ``Failure`` results are self-explanatory. The ``Undecided`` result means that the validator was unable to determine whether the instance is valid or not. This can happen if the validator was unable to retrieve the profile, or if the profile contains errors. The property ``IsSuccessful`` can be used to quickly determine whether the validation was successful or not.

While validating the instance, the validator will collect *evidence*, which is a list of warnings, errors and trace information. The report has an ``Evidence`` property that contains this full list, while the ``Errors`` and ``Warnings`` properties can be used to quickly filter on just the error and warnings. Each such error or warning is represented using an ``IssueAssertion``, which contains (amongst other information) the human-readable message, an issue number and the location in the instance where the issue was found. The issue number is an integer that corresponds to the code for an ``Issue`` as found in the ``Hl7.Fhir.Support`` namespace. This makes it easy to test for specific kind of errors:

.. code-block:: csharp

  var result = validator.Validate(p, Canonical.ForCoreType("Patient").ToString());
  result.Errors.Should().ContainSingle().Which.IssueNumber.Should().Be(Issue.CONTENT_ELEMENT_CHOICE_INVALID_INSTANCE_TYPE.Code);
  result.IsSuccessful.Should().BeFalse();

If you run multiple validations (for example, when you want to validate the same resource against several profiles), you can combine the results using the ``ResultReport.Combine()`` into a single report.

Finally, the extension method ``ToOperationOutcome()`` can be used to convert the validation results into an ``OperationOutcome`` resource, which can be serialized and returned to the client when working in a RESTful environment.


External references
-------------------
FHIR instances can contain references to other resources, which can be used to validate the instance. For example, a Patient resource can contain a reference to a Practitioner. The validator can be configured with an ``IExternalReferenceResolver`` to retrieve these resources. If this service is not supplied, the validator will simply not try to validate the resources that these references point to.

The interface looks like this:

.. code-block:: csharp

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

When implementing this interface, you can return either a ``Resource`` or an ``ElementNode``, depending on whether you are working with POCO's or ``ITypedElement``-based models. Return ``null`` if the reference cannot be resolved.

Selecting profiles to validate against
--------------------------------------

In the ``Validate()`` call in the examples above, we passed an explicit profile url to validate against. This is not always necessary. If you leave out the profile url, the validator will try to find a profile url in the ``Meta`` of the instance. If it finds one, it will validate against that profile. If it does not find one, it will validate against the "default" core profile for the resource type. You would normally only pass in an explicit profile url if you want to validate against a specific profile, e.g. one for US Core or other national profiles. 

The behaviour of following the profiles in Meta can be changed by setting the ``MetaProfileSelector`` property.

Other configuration operations
------------------------------
Several other properties of the Validator can be configured to change the behaviour of the validator, even between calls to the ``Validate()`` method. These properties are:

.. list-table::
   :header-rows: 1

   * - Property
     - Use
   * - ValidateBestPracticesSeverity
     - Determines how to deal with failures of FhirPath constraints marked as "best practice". Default is ``Warning``
   * - MetaProfileSelector
     - Determines which profiles from a Resource's ``Meta`` to validate the instance against. Default is to use all profiles in ``Meta``.
   * - ExtensionUrlFollower
     - Determines what do do when an extension is encountered. If not set, then a validation of an Extension will warn if the extension cannot be resolved, or will return an error when the extension cannot be resolved and is a modififier extension.
   * - TypeNameMapper
     - A function that maps a type name found in ``TypeRefComponent.Code`` to a resolvable canonical. If not set, it will prefix the type with the standard ``http://hl7.org/fhir/StructureDefinition`` prefix.
   * - SkipConstraintValidation
     - Enables or disables the validation of FhirPath constraints. Default is ``false``.

Full Example
------------
We have created a full example that shows how to use the validator using a terminology service and the FHIR core package resolvers. See `this GitHub repo <https://github.com/FirelyTeam/Firely.Fhir.ValidationDemo>`_ for more information.

