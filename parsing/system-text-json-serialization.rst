=============================================
Serialization with POCOs and System.Text.Json
=============================================

With the introduction of the NET5.0 target of the SDK, we have added a serializer based on the new ``System.Text.Json`` namespace. 
These serializers serialize POCO's directly to ``Utf8JsonWriter`` objects, achieving higher throughput than the existing serializers with a much smaller memory footprint.

The functionality is exposed through a ``JsonFhirConverter``. To use it, set up a new ``JsonSerializerOptions`` to add this converter, and then call the ``JsonSerializer``:

.. code-block:: csharp

    Patient p = new() {  };
    var options = new JsonSerializerOptions().ForFhirCompact(typeof(Patient).Assembly);
    string patientJson = JsonSerializer.Serialize(p, options);

The ``ForFhirCompact()`` method initializes the options to use the FHIR Json converter, there is a ``ForFhirPretty()`` too. 
Both methods take an ``Assembly`` as an argument, which is the assembly where the SDK's POCO classes can be found. This is irrelevant
for serialization, but is used by deserializer to locate the classes to instantiate when parsing FHIR data. More details can be found
on the section on deserialization.

Note that the serializer will *not* validate the data passed to it, so it can easily be driven to produce Json that is incorrect, e.g.
this code:

.. code-block:: csharp

    FhirBoolean b = new() { ObjectValue = "treu" };
    Patient p = new() { Contact = new() { new Patient.ContactComponent() }, ActiveElement = b };

will produce the following invalid FHIR Json:

.. code-block:: json

    {
      "resourceType": "Patient",
      "active": "treu",
      "contact": [{}]
    }

This is great for round-tripping, and keeping validation duties out of the serializer avoids duplicate work and helps performance, but it also means one
should use the validation capabilities of the SDK before serialization is done.

If you do not want to set up a converter, it is possible to invoke the serializer directly, using the 
``SerializeToFhirJson(this IReadOnlyDictionary<string, object> members, Utf8JsonWriter writer)`` extension method found in ``JsonFhirDictionarySerializer``.

