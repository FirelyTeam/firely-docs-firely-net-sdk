.. _systemtextjsondeserialization:

===============================================
Deserialization with POCOs and System.Text.Json
===============================================

With the introduction of the NET 5.0 target of the SDK, we have added a deserializer based on the new ``System.Text.Json`` namespace.
These deserializers deserialize POCO's directly from ``Utf8JsonReader`` objects, achieving higher throughput than the existing serializers with a much smaller memory footprint.

The functionality is exposed through ``JsonFhirConverter``, which is an implementation of ``System.Text.Json``'s ``JsonConvert<T>``.
To use it, set up a new ``JsonSerializerOptions`` to add this converter, and then call the ``JsonSerializer``:

.. code-block:: csharp

    var jsonInput = "{.....}";
    var options = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector);
    var patient = JsonSerializer.Deserialize<Patient>(jsonInput, options);

The ``ForFhir()`` method initializes the options to use the FHIR Json converter. This methods has an overload that takes a ``ModelInspector`` as an argument,
which is the metadata about where the SDK's POCO classes can be found and which will be used to create the resource encountered in the json input text. If you are working
with one specific version of FHIR (i.e. you are using a NuGet assembly for R4), there will be an overload
that does not require the ``ModelInspector`` argument, and it will default to the version of FHIR you have included in your project.

NB: It is very important that you reuse instances of ``JsonSerializerOptions`` across all your deserialization calls, otherwise performance will degrade tremendously.

Finding errors while deserializing
----------------------------------
When calling `JsonSerializer.Deserialize()`, the SDK will throw a ``DeserializationFailedException`` when it encounters errors, so it might be wise to add some error handling:

.. code-block:: csharp

    string patientString = "{\"resourceType\": \"Patient\",\"id\": \"example-patient\"}";
    var options = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector);
    try
    {
        Patient p = JsonSerializer.Deserialize<Patient>(patientString, options);
    }
    catch (DeserializationFailedException e)
    {
        Console.WriteLine(e.Message);
    }

This exception has two properties:

* ``Exceptions`` - A list of ``CodedException``, which carries both a human-readable error and a code to be used for automated handling of errors.
* ``PartialResult`` - A partly constructed POCO, containing as much data as was possible to deserialize from the input text.

The ``PartialResult`` allows you to access data from the input text, even if the SDK encountered errors, so you can display useful (debug) information to the user,
or even continue working with the result, despite the errors. Be aware though, that data maybe lost when errors have been encountered.

When no exception is thrown, the returned POCO can be considered complete and structurally valid. Note however, that FHIR allows you to specify extensive validation rules using
profiles, and the deserializer does not perform profile validation, just basic structural validation (e.g. cardinalities). See :ref:`poco validation<poco-validation>`
for more information.

The ``Exceptions`` property will contain a list of ``CodedException``, and these might contain a mix of exceptions encountered by the parser (``FhirJsonException``) and the validator (``CodedValidationException``). These exceptions are coded, so they do not only contain a human-readable message, but a persistent code as well. The codes are defined as constants on the ``FhirJsonException`` class. Rather than maintaining a (probably outdated) table of codes here, it is preferable to glance the current set `from the code <https://github.com/FirelyTeam/firely-net-common/blob/develop/src/Hl7.Fhir.Support.Poco/Serialization/FhirJsonException.cs>`_.


Configuration options for the deserializer
------------------------------------------
The ``ForFhir()`` method can take an argument of type ``FhirJsonPocoDeserializerSettings`` which allows changing the default behaviour of the deserializer:

* You can indicate that you do not want to deserialize base64 encoded data - which will decrease both memory consumption and increase parsing speed.
  The original string text containing the base64 data can be retrieved (and subsequently decoded) using the elements ``ObjectValue`` property.
* You can pass in a callback that is called when a primitive json value fails to parse. Using this callback, you can customize what happens when the deserializer
  encounters a primitive value that does not strictly adhere to FHIR's rules and regexes for the primitive datatypes.
* You can set a validator to use, which is called just after a value is parsed and just before it is used to set a property on the POCO. By default this setting
  contains an instance of the ``DataAnnotationDeserialzationValidator``, which validates the values according to the validation attributes specified on the element
  in the generated POCO, and which will generally validate the basic structural validations provided by FHIR. See :ref:`poco validation<poco-validation>` for more
  information about POCO validation with data attributes.

The ``FhirJsonPocoDeserializerSettings`` class also contains static methods for ignoring specific errors, which can be useful at certain occasions, like when you know that certain errors are not relevant to your use case:

The ``UsingMode()`` method takes a member of enum ``DeserializerMode`` as its argument and configures the deserializer to ignore all errors of a certain type. Four modes are defined in the enum:

* ``Strict`` - Ignores no issues. This is the default mode.
* ``Recoverable`` - Ignores issues which do NOT lead to data loss. Recoverable issues mean that all data present in the parsed data could be retrieved and captured in the POCO model, even if the syntax or the data was not fully FHIR compliant.
* ``BackwardCompatible`` - Ignores issues which are allowable for backwards compatibility. An issue is allowable for backwards compatibility if it could be caused because an older parser encounters data coming from a newer FHIR release. This means allowing unknown elements, codes and types in a choice element. Note that the POCO model cannot capture these newer elements and data, so this means data loss may occur.
* ``Ostrich`` - Ignores all issues, including those that lead to data loss. This mode is useful for either debugging purposes, or when you are sure that the input data is correct and you want to ignore all issues.

To further customize the list of ignored issues, you can use the following static extension methods:

* ``Ignoring()`` - Ignores a list of issues, identified by their codes.
* ``Enforcing()`` - Enforces a list of issues, identified by their codes. This method is useful when you want to enforce a certain issue, even when the deserializer is in a mode that would normally ignore it.

Note that ``Ignoring()`` and ``Enforcing()`` are both left-associative, and that the order of the calls to UsingMode and Ignoring/Enforcing is important, as setting a new mode will override any previous Ignoring/Enforcing calls. For example:

.. code-block:: csharp

    // setting mode overrides previous calls to Ignoring/Enforcing

    // will enforce the issue, even though the deserializer is in ostrich mode
    _ = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector)
        .UsingMode(DeserializerMode.Ostrich).Enforcing([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE]);

    // will ignore the issue, even though it was enforced before, because the mode is set after the enforcing call
    _ = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector)
        .Enforcing([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE]).UsingMode(DeserializerMode.Ostrich);


.. code-block:: csharp

    // ignoring/enforcing are left-associative

    // will enforce the issue, even though it was ignored before
    _ = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector)
        .Ignoring([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE]).Enforcing([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE]);

    // will ignore the issue, even though it was enforced before
    _ = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector)
        .Enforcing([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE]).Ignoring([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE]);

    // will ignore both issues (but should be avoided, as the second call to Ignoring could be replaced by a single call with both issues)
    _ = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector)
        .Ignoring([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE]).Ignoring([FhirJsonException.RESOURCETYPE_SHOULD_BE_STRING_CODE]);

    // will ignore both issues
    _ = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector)
        .Ignoring([FhirJsonException.EXPECTED_START_OF_OBJECT_CODE, FhirJsonException.RESOURCETYPE_SHOULD_BE_STRING_CODE]);

The error codes are defined as constants on the ``FhirJsonException`` class. Rather than maintaining a (probably outdated) table of codes here, it is preferable to glance the current set `from the code <https://github.com/FirelyTeam/firely-net-sdk/blob/develop/src/Hl7.Fhir.Base/Serialization/FhirJsonException.cs>`_.

