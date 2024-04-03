=============================================
Serialization with POCOs and System.Text.Json
=============================================

With the introduction of the NET 5.0 target of the SDK, we have added a serializer based on the new ``System.Text.Json`` namespace. 
These serializers serialize POCO's directly to ``Utf8JsonWriter`` objects, achieving higher throughput than the existing serializers with a much smaller memory footprint.

The functionality is exposed through ``JsonFhirConverter``, which is an implementation of ``System.Text.Json``'s ``JsonConvert<T>``.
To use it, set up a new ``JsonSerializerOptions`` to add this converter, and then call the ``JsonSerializer``:

.. code-block:: csharp

    Patient p = new() {  };
    var options = new JsonSerializerOptions().ForFhir(ModelInfo.ModelInspector).Pretty();
    string patientJson = JsonSerializer.Serialize(p, options);

The ``ForFhir()`` method initializes the options to use the FHIR Json converter. This methods has an overload that takes a ``ModelInspector`` 
as an argument, which is the metadata about where the SDK's POCO classes can be found. This
determines which version of FHIR to use for serialization and is used by deserializer to locate the classes to instantiate when parsing
FHIR data. If you are working with one specific version of FHIR (i.e. you are using a NuGet assembly for R4), there will be an overload
that does not require the ``ModelInspector`` argument, and it will default to the version of FHIR you have included in your project.

NB: It is very important that you reuse instances of ``JsonSerializerOptions`` across all your deserialization calls, otherwise performance will degrade tremendously.

More details can be found on the section on :ref:`deserialization<systemtextjsondeserialization>`.

Note that the serializer will *not* validate the data passed to it, so it can easily be driven to produce Json that is incorrect, e.g.
this code:

.. code-block:: csharp

    FhirBoolean b = new() { ObjectValue = "treu" };
    Patient p = new() { Contact = new() [ new Patient.ContactComponent() ], ActiveElement = b };

will produce the following invalid FHIR Json:

.. code-block:: json

    {
      "resourceType": "Patient",
      "active": "treu",
      "contact": [{}]
    }

This is by design, as this is useful for round-tripping incorrect stored historical data. It avoids duplicate validation work and increases performance as well.
To ensure correct json output, you should :ref:`validate<validation>` the instances first.

If you do not want to set up a converter, it is possible to invoke the serializer directly by
instantiating a ``JsonFhirDictionarySerializer``, and then calling its ``Serialize()`` method. The ``JsonFhirDictionarySerializer`` can be subclassed
to change the way primitives are serialized in the virtual method ``SerializePrimitiveValue()``. This will become especially useful in .NET 6, which adds
functionality to allow the user to write "raw" json to the output stream. An example is overriding the handling of very large decimals.

Generating a summary
--------------------
The serializer can be told to serialize only parts of an object's tree, thus generating a `summary view <http://hl7.org/fhir/search.html#summary>`_ of the FHIR data`.

To enable summary generation, create a concrete subclass of the ``SerializationFilter`` class, there are several pre-defined factory methods that you can use:

* SerializationFilter.ForSummary() - returns a new serialization filter to support ``_summary=true``
* SerializationFilter.ForText() - returns a new serialization filter for ``_summary=text``
* SerializationFilter.ForData() - returns a new serialization filter for ``_summary=data``
* SerializationFilter.ForCount() - returns a new serialization filter for ``_summary=count``
* SerializationFilter.ForElements() - returns a new serialization filter for ``_elements=``

The other summary forms mentioned in the FHIR specification do not need specific support from the serializer and can be constructed by hand.

Once created, such a filter must then be passed to one of the serialization methods, like so:

.. code-block:: csharp

    var summaryFilter = SerializationFilter.ForSummary();
    var options = new JsonSerializerOptions().ForFhir(typeof(Patient).Assembly, summaryFilter).Pretty();

The filters are highly configurable and it is possible to write your own by subclassing ``SerializationFilter``. Please refer to the (pretty trivial)
implementation of the current summary filters (e.g. ``ElementMetadataFilter.cs`` and ``BundleFilter.cs``) in the source code for more information.