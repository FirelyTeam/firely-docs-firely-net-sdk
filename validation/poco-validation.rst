.. _poco-validation:

=====================================
Validating POCOs with DataAnnotations
=====================================

The POCOs generated in the ``Hl7.Fhir.Core`` assembly are annotated with custom
`ValidationAttributes <https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute>`_ based on the standard .NET
validation framework from the ``System.ComponentModel.DataAnnotations`` namespace. These attributes validate primitive datatypes to make sure you use the
correct format for datetimes, adhere to the correct cardinalities for repeating elements etc.

For example the ``AllowedTypes`` attribute below specifies that the allowed choices for ``ChargeItem.occurrence`` are ``dateTime`` and ``timing``. Since this attribute derives from ``ValidationAttribute``, it takes part in validation and will at runtime validate that the ``Occurrence`` property is indeed assigned one of these types.

.. code-block:: csharp

    [FhirElement("occurrence", InSummary=true, Order=160, Choice=ChoiceType.DatatypeChoice)]
    [AllowedTypes(typeof(Hl7.Fhir.Model.FhirDateTime),typeof(Hl7.Fhir.Model.Period),typeof(Hl7.Fhir.Model.Timing))]
    public Hl7.Fhir.Model.DataType Occurrence
    {
    }

The following table lists the attributes used by the SDK:

.. list-table:: Available attribute validations
   :widths: 10 90
   :header-rows: 1

   * - Attribute
     - Validation performed
   * - AllowedTypesAttribute
     - Applied to "choice" properties of type ``DataType`` (like ``Observation.value[x]``), this attribute verifies whether the type of the instance data in the property is one of the allowed types for the choice.
   * - CardinalityAttribute
     - Applied to a property of type ``List<T>``, this attribute verifies whether the number of items in the list conforms to the stated cardinality for the element.
   * - CanonicalPatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``canonical``.
   * - CodePatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``code``.
   * - DatePatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``date``.
   * - DateTimePatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``dateTime``.
   * - IdPatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``id``.
   * - InstantPatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``id``.
   * - NarrativeXhtmlPattern
     - Verifies whether the string property (normally inside Div) in ``Div`` is valid XML and (if configured to do so), validates whether this XML adheres to the `additional rules for FHIR narrative <https://www.hl7.org/fhir/narrative.html>`_.
   * - OidPatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``oid``.
   * - TimePatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``time``.
   * - UriPatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``uri``.
   * - UuidPatternAttribute
     - Verifies whether the string property is a correctly formatted FHIR ``uuid``.

In addition, the POCO classes implement `IValidatableObject <https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.ivalidatableobject>`_ to perform additional validations not bound to a single property:

.. list-table:: Additional non-attribute validations
   :widths: 10 90
   :header-rows: 1

   * - POCO
     - Validation performed
   * - *All resources*
     - Validates that contained resources do not themselves have nested contained resources
   * - Code<T>
     - Validates that the string in ``ObjectValue`` can be parsed as one of the enum values in the enumeration ``T``.

As is clear from the set of validations described above, the attribute-based validation does not cover all constraints that can be expressed using a ``StructureDefinition``, not even all those for the non-profiled FHIR core datatypes. Notable constrains *not* checked are:

* **Terminology** - the attribute validator will not call upon a terminology service to validate the coded datatypes, it will however validate correct values for all required, explicit bindings, since these are generated as .NET enumerations.
* **Slices** - these are not used in the definitions of the core resources from which the POCOs are generated. There is also no trivial method to express these constraints in a .NET class model, so attribute validation does not cover them.
* **FhirPath invariants** - FHIR model definitions generally contain additional invariants specified using `FhirPath <http://hl7.org/fhirpath/>`_. These are not generated into the POCO classes and require an external FhirPath engine.

How to invoke attribute validation
----------------------------------
Attribute can be invoked using the familiar validation methods from ``System.ComponentModel.DataAnnotations``, so it is possible to call ``Validator.ValidateObject()`` or ``IValidatableObject.Validate()`` directly. However, the attributes mentioned in this section use specific extensions on the ``ValidationContext`` to work properly, so the SDK provides an extension method  ``Validation.Validate(this Base, ...)`` that can be used to ensure this context is properly instantiated:

.. code-block:: csharp

    using Hl7.Fhir.Validation;

    var patient = new Patient() {...}
    patient.Validate(recurse: true, narrativeValidation: NarrativeValidationKind.FhirXhtml);

When the instance is invalid, this method will throw an Exception detailing the validation problem. All SDK validation attributes will throw an Exception of type ``CodedValidationException``, which not only carries a message for the exception for human consumption, but also a pertinent code that can be used to recognize programmatically. These codes can be found as constants on ``CodedValidationException``. Examples are:

.. list-table:: Examples of coded exceptions
   :widths: 10 30 60
   :header-rows: 1

   * - Constant
     - Code
     - Message
   * - PVAL101
     - CHOICE_TYPE_NOT_ALLOWED_CODE
     - "Value is of type '{0}', which is not an allowed choice."
   * - PVAL107
     - DATE_LITERAL_INVALID_CODE
     - "'{0}' is not a correct literal for a date."
   * - PVAL117
     - CONTAINED_RESOURCES_CANNOT_BE_NESTED_CODE
     - "It is not allowed for a resource to contain resources which themselves contain resources."

The full set of codes can be found `in the source code <https://github.com/FirelyTeam/firely-net-common/blob/develop/src/Hl7.Fhir.Support.Poco/Validation/CodedValidationException.cs>`_

The :ref:`new deserializer<systemtextjsondeserialization>` invokes the attribute-based validation described here while performing deserialization.
This means that, after you have deserialized an object, there is no need to invoke validation yourself, and validation results should be consistent
between deserialized POCOs and POCOs that are constructed in code.
