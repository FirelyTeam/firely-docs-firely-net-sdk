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
    var options = new JsonSerializerOptions().ForFhir(typeof(Patient).Assembly);
    var patient = JsonSerializer.Deserialize<Patient>(jsonInput, options);

The ``ForFhir()`` method initializes the options to use the FHIR Json converter. This methods has an overload that takes an ``Assembly`` as an argument,
which is the assembly where the SDK's POCO classes can be found and which will be used to create the resource encountered in the json input text. If you are working
with one specific version of FHIR (i.e. you are using a NuGet assembly for R4), there will be an overload
that does not require the ``Assembly`` argument, and it will default to the version of FHIR you have included in your project.

NB: It is very important that you reuse instances of ``JsonSerializerOptions`` across all your deserialization calls, otherwise performance will degrade tremendously.

Finding errors while deserializing
----------------------------------
When calling `JsonSerializer.Deserialize()`, the SDK will throw a ``DeserializationFailedException`` when it encounters errors, so it might be wise to add some error handling:

.. code-block:: csharp

    string patientString = "{\"resourceType\": \"Patient\",\"id\": \"example-patient\"}";
    var options = new JsonSerializerOptions().ForFhir(typeof(Patient).Assembly);
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
