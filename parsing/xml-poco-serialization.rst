=============================================
Serialization with POCOs and XmlWriter
=============================================

With the ``FhirXmlPocoDeserializer`` have added a serializer that serializes POCO's directly to ``XmlWriter`` objects, achieving higher throughput than the existing serializers with a much smaller memory footprint.

To use it, we first create an ``Xmlwriter`` and a ``FhirXmlPocoSerializer``, and then use the ``Serialize`` function to serialize a POCO to the XmlWriter. Which you can later use to get you the actual xml string.

.. code-block:: csharp

    Patient p = new() {  };
    var sb = new StringBuilder();
    using (var w = XmlWriter.Create(sb))
    {   
       var serializer = new FhirXmlPocoSerializer(Specification.FhirRelease.STU3);
       serializer.Serialize(p, w);
    }
    var xmlString = sb.ToString();

When initializing the ``FhirXmlPocoSerializer`` you have to specify which FHIR version you are using for serialization.

Note that the serializer will *not* validate the data passed to it, so it can easily be driven to produce XML that is incorrect, e.g.
this code:

.. code-block:: csharp

    FhirBoolean b = new() { ObjectValue = "treu" };
    Patient p = new() { Contact = new() [ new Patient.ContactComponent() ], ActiveElement = b };

will produce the following invalid FHIR Xml:

.. code-block:: xml

    <Patient xmlns="http://hl7.org/fhir">
        <active value = "treu"/>
        <contact></contact>
    </Patient>

This is by design, as this is useful for round-tripping incorrect stored historical data. It avoids duplicate validation work and increases performance as well.
To ensure correct xml output, you should :ref:`validate<validation>` the instances first.

Generating a summary
--------------------
The serializer can be told to serialize only parts of an object's tree, thus generating a `summary view <http://hl7.org/fhir/search.html#summary>`_ of the FHIR data`.

To enable summary generation, create a concrete subclass of the ``SerializationFilter`` class, there are several pre-defined factory methods that you can use:

* SerializationFilter.ForSummary() - returns a new serialization filter to support ``_summary=true``
* SerializationFilter.ForText() - returns a new serialization filter for ``_summary=text``
* SerializationFilter.ForData() - returns a new serialization filter for ``_summary=data``
* SerializationFilter.ForElements() - returns a new serialization filter for ``_elements=``

The other summary forms mentioned in the FHIR specification do not need specific support from the serializer and can be constructed by hand.

Once created, such a filter must then be passed to the serialize method, like so:

.. code-block:: csharp

    var summaryFilter = SerializationFilter.ForSummary();
    Patient p = new() {  };
    var sb = new StringBuilder();
    using (var w = XmlWriter.Create(sb))
    {   
       var serializer = new FhirXmlPocoSerializer(Specification.FhirRelease.STU3);
       serializer.Serialize(p, w, summaryFilter);
    }

The filters are highly configurable and it is possible to write your own by subclassing ``SerializationFilter``. Please refer to the (pretty trivial)
implementation of the current summary filters (e.g. ``ElementMetadataFilter.cs`` and ``BundleFilter.cs``) in the source code for more information.