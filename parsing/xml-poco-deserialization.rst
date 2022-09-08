.. _xmlpocodeserialization:

===============================================
Deserialization with POCOs and XmlReader
===============================================

With the ``FhirXmlPocoDeserializer`` have added a deserializer that deserializes POCO's directly from ``XmlReader`` objects, achieving higher throughput than the existing serializers with a much smaller memory footprint.

To use it, we first create an ``XmlReader`` and a ``FhirXmlPocoDeserializer`` and then use the ``Deserialize`` function to deserialize the xml to the desired POCO:

.. code-block:: csharp

    var xml = "<Patient xmlns=\"http://hl7.org/fhir\"> .. </Patient>";
    var xmlReader = XmlReader.Create(new StringReader(xml))
    var deserializer = new FhirXmlPocoDeserializer(typeof(TestPatient).Assembly)
    var patient = deserializer.Deserialize<Patient>(xml);

You can see that the ``FhirXmlPocoDeserializer`` takes an ``Assembly`` as an argument, which is the assembly where the SDK's POCO classes can be found and which will be used to create the resource encountered in the xml input text.

Finding errors while deserializing
----------------------------------
When calling `Deserialize()`, the SDK will throw a ``DeserializationFailedException`` when it encounters errors, so it might be wise to add some error handling:

.. code-block:: csharp

    var xml = "<Patient xmlns=\"http://hl7.org/fhir\"> .. </patient>";
    var xmlReader = XmlReader.Create(new StringReader(xml))
    var deserializer = new FhirXmlPocoDeserializer(typeof(TestPatient).Assembly)
  
    try
    {
       var patient = deserializer.Deserialize<Patient>(xml);
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

The ``Exceptions`` property will contain a list of ``CodedException``, and these might contain a mix of exceptions encountered by the parser (``FhirXmlException``) and the validator (``CodedValidationException``). These exceptions are coded, so they do not only contain a human-readable message, but a persistent code as well. The codes are defined as constants on the ``FhirXmlException`` class. Rather than maintaining a (probably outdated) table of codes here, it is preferable to glance the current set `from the code <https://github.com/FirelyTeam/firely-net-common/blob/develop/src/Hl7.Fhir.Support.Poco/Serialization/FhirXmlException.cs>`_.


Configuration options for the deserializer
------------------------------------------
The ``Deserialize()`` method can take an argument of type ``FhirXmlPocoDeserializerSettings`` which allows changing the default behavior of the deserializer:

* You can indicate that you do not want to deserialize base64 encoded data - which will decrease both memory consumption and increase parsing speed.
  The original string text containing the base64 data can be retrieved (and subsequently decoded) using the elements ``ObjectValue`` property.
* You can set a validator to use, which is called just after a value is parsed and just before it is used to set a property on the POCO. By default this setting
  contains an instance of the ``DataAnnotationDeserialzationValidator``, which validates the values according to the validation attributes specified on the element
  in the generated POCO, and which will generally validate the basic structural validations provided by FHIR. See :ref:`poco validation<poco-validation>` for more
  information about POCO validation with data attributes.
