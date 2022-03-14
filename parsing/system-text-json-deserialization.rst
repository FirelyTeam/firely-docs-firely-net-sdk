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

Finding errors while deserializing
----------------------------------

(on errors)

Note that the serializer will *not* validate the data passed to it, so it can easily be driven to produce Json that is incorrect, e.g.
this code:

Configuration options for the deserializer
------------------------------------------
The ``ForFhir()`` method can take an argument of type ``FhirJsonPocoDeserializerSettings`` which allows changing the default behaviour of the deserializer:

* You can indicate that you do not want to deserialize base64 encoded data - which will decrease both memory consumption and increase parsing speed. The original string text containing the base64 data can be retrieved (and subsequently decoded) using the elements ``ObjectValue`` property.
* You can pass in a callback that is called when a primitive json value fails to parse. Using this callback, you can customize what happens when the deserializer encounters a primitive value that does not strictly adhere to FHIR's rules and regexes for the primitive datatypes.
* You can set a validator to use, which is called just after a value is parsed and just before it is used to set a property on the POCO. By default this setting contains and instance of the ``DataAnnotationDeserialzationValidator``, which validates the values according to the validation attributes specified on the element in the generated POCO, and which will generally validate the basic structural validations provided by FHIR. See XXXXXX (link to the section that is currently a PR) for more information about POCO validation with data attributes.




