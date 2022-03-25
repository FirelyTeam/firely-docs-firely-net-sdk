.. _systemtextjsondeserialization:

===============================================
Deserialization with POCOs and System.Text.Json
===============================================

Deserialization is handled in the same way as Serialization with ``System.Text.Json``. To use it, set up a new ``JsonSerializerOptions`` to add the ``JsonFhirConverter``, and then call the ``JsonSerializer``:

.. code-block:: csharp

    string patientString = "{\"resourceType\": \"Patient\",\"id\": \"example-patient\"}";
    var options = new JsonSerializerOptions().ForFhir(typeof(Patient).Assembly);
    Patient p = JsonSerializer.Deserialize<Patient>(patientString, options);

You can, of course, also use the async version:

.. code-block:: csharp

    string patientString = "{\"resourceType\": \"Patient\",\"id\": \"example-patient\"}";
    var options = new JsonSerializerOptions().ForFhir(typeof(Patient).Assembly);
    Patient p = await JsonSerializer.DeserializeAsync<Patient>(patientString, options);

The ``ForFhir()`` method initializes the options to use the FHIR Json converter. This method has an overload that takes an ``Assembly`` as an argument, which is the assembly where the SDK's POCO classes can be found. This
determines which version of FHIR to use for serialization and is used by deserializer to locate the classes to instantiate when parsing
FHIR data. If you are working with one specific version of FHIR (i.e. you are using a NuGet assembly for R4), there will be an overload
that does not require the ``Assembly`` argument, and it will default to the version of FHIR you have included in your project.

The deserializer also validates the JSON data and will throw a ``Hl7.Fhir.Serialization.DeserializationFailedException`` if the data is invalid. It might be wise to add some error handling:

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