.. _profile-validation:

================================
Validating data against profiles
================================

The profile validator is distributed as a separate package, and can be found on NuGet, by searching for ``Firely.Fhir.Validation.<spec version>``. The main class in this package is the `Validator`. To construct a new instance, we need to supply it with two required dependencies:

* A reference to a terminology service (see :ref:`terminology-service`), specifically a ``ICodeValidationTerminologyService``. This service is needed to validate the codes used in the instance against the code systems and value sets specified in the profile.
* A reference to a resource resolver (see :ref:`package-source`), specifically an ``IAsyncResourceResolver``. This service allows the validator to retrieve the profiles that are referenced by the profile being validated.

The last dependency is optional, but highly recommended, and that is an ``IExternalReferenceResolver``. This service allows the validator to retrieve the resources that are referenced by the instance being validated. If this service is not supplied, the validator will simply not try to validate the resources that these references point to.

.. code-block:: csharp

    var packageServer = "https://packages.simplifier.net";
    var fhirRelease = FhirRelease.STU3;
    var packageResolver = FhirPackageSource.CreateCorePackageSource(ModelInfo.ModelInspector, fhirRelease, packageServerUrl);
    var resourceResolver = new CachedResolver(packageResolver);
    var terminologyService = new LocalTerminologyService(resourceResolver);

    var validator = new Validator(terminologyService, resourceResolver);
    var testOrganization = new Organization { };

    var result = validator.Validate(testOrganization, "http://hl7.org/fhir/StructureDefinition/Organization");

The above code initializes a package resolver, which is configured to retrieve the basic core FHIR STU3 packages. It also creates a terminology service, which is configured to use the same resource resolver to retrieve the code systems and value sets that are referenced by the profiles. Finally, it creates a validator, and runs a validation against a new instance of the Organization resource, using the Organization profile.

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
TODO

Other configuration operations
------------------------------

* FhirPathCompiler
*         /// <summary>
        /// Determines how to deal with failures of FhirPath constraints marked as "best practice". Default is <see cref="ValidateBestPracticesSeverity.Warning"/>.
        /// </summary>
        /// <remarks>See <see cref="FhirPathValidator.BestPractice"/>, <see cref="ValidateBestPracticesSeverity"/> and
        /// https://www.hl7.org/fhir/best-practices.html for more information.</remarks>
        public ValidateBestPracticesSeverity ConstraintBestPractices = ValidateBestPracticesSeverity.Warning;

        /// <summary>
        /// The <see cref="MetaProfileSelector"/> to invoke when a <see cref="Meta.Profile"/> is encountered. If not set, the list of profiles
        /// is used as encountered in the instance.
        /// </summary>
        public MetaProfileSelector? MetaProfileSelector = null;

        /// <summary>
        /// The <see cref="Validation.ExtensionUrlFollower"/> to invoke when an <see cref="Extension"/> is encountered in an instance.
        /// If not set, then a validation of an Extension will warn if the extension cannot be resolved, or will return an error when 
        /// the extension cannot be resolved and is a modififier extension.
        /// </summary>
        public ExtensionUrlFollower? ExtensionUrlFollower = null;

        /// <summary>
        /// A function that maps a type name found in <c>TypeRefComponent.Code</c> to a resolvable canonical.
        /// If not set, it will prefix the type with the standard <c>http://hl7.org/fhir/StructureDefinition</c> prefix.
        /// </summary>
        public TypeNameMapper? TypeNameMapper { get; set; }

        /// <summary>
        /// StructureDefinition may contain FhirPath constraints to enfore invariants in the data that cannot
        /// be expresses using StructureDefinition alone. This validation can be turned off for performance or
        /// debugging purposes. Default is 'false'.
        /// </summary>
        public bool SkipConstraintValidation { get; set; } // = false;



