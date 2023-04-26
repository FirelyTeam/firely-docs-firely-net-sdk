Creating a FhirClient
---------------------

Before we can do any of the interactions explained in the next part, we
have to create a new ``FhirClient`` instance. This is done by passing the url of the
FHIR server's endpoint as a parameter to the constructor:

.. code:: csharp

    var client = new FhirClient("http://server.fire.ly");

The constructor method is overloaded, to enable you to use a URI instead of a string.
As second parameter to the constructor, you can specify whether the client should
perform a conformance check to see if the server has got a compatible FHIR version.
The default setting is ``false``.

A FhirClient works with a single server. If you work with multiple servers simultanuously, you'll
have to create a FhirClient for each of them. Since resources may reference other resources on a 
different FHIR server, you'll have to inspect any references and direct them to the right FhirClient.
Of course, if you're dealing with a single server within your organization or a single
cloud-based FHIR server, you don't have to worry about this. Note: the FhirClient is not thread-safe,
so you will need to create one for each thread, if necessary. But don't worry: creating an instance
of a FhirClient is cheap, the connection will not be opened until you start working with it.

There's a list of `publicly available test 
servers <http://wiki.hl7.org/index.php?title=Publicly_Available_FHIR_Servers_for_testing>`__ you can use.

.. _sdk-minimal:

FhirClient communication options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To specify some specific settings, you add a ``FhirClientSettings`` to the constructor:

.. code:: csharp

	var settings = new FhirClientSettings
            {
                Timeout = 0,
                PreferredFormat = ResourceFormat.Json,
                VerifyFhirVersion = true,
                ReturnPreference = ReturnPreference.Minimal
            };
            
    var client = new FhirClient("http://server.fire.ly", settings)

You can also toggle these settings after the client has been initialized.

To specify the preferred format --JSON or XML-- of the content to be used when communicating
with the FHIR server, you can use the ``PreferredFormat`` attribute:

.. code:: csharp

	client.Settings.PreferredFormat = ResourceFormat.Json;

The FHIR client will send all requests in the specified format. Servers
are asked to return responses in the same format, but may choose
to ignore that request. The default setting for this field is ``XML``.

When communicating the preferred format to the server, this can either be done by appending
``_format=[format]`` to the URL, or setting the ``Accept`` HTTP header. The client uses the
latter by default, but if you want, you can use the ``_format`` parameter instead:

.. code:: csharp

	client.Settings.UseFormatParameter = true;

If you perform a ``Create``, ``Update`` or ``Transaction`` interaction, you can request the server
to either send back the complete representation of the interaction, or a minimal data set, as
described in the `Managing Return Content <http://www.hl7.org/fhir/http.html#2.21.0.5.2>`_ section
of the specification. The default setting is to ask for the complete representation. If you want to
change that request, you can set the ``PreferredReturn`` attribute:

.. code:: csharp

	client.Settings.ReturnPreference = ReturnPreference.Minimal;
	
This sets the ``Prefer`` HTTP header in the request to ``minimal``, asking the
server to return no body in the response.

You can set the timeout to be used when making calls to the server with the ``Timeout`` attribute:

.. code:: csharp

	client.Timeout = 120000; // The timeout is set in milliseconds, with a default of 100000

Selecting a serializer
^^^^^^^^^^^^^^^^^^^^^^
As described in the section on :ref:`serialization <FHIR-parsing>`, there are two parser families: the legacy XML and Json parsers that have been 
in the SDK for years, and the improved ones, introduced in SDK 4. Each of these parsers can be configured
to ignore certain kind of errors (for example, to allow unknown elements). It is possible to configure the 
``FhirClient`` to use the serializer family of your chosing. Here is an example, where we configure the FhirClient
to use the newer serializer (and deserializer), in "strict" mode (refusing all syntactical errors in the json):

.. code:: csharp
          
    var client = new FhirClient("http://server.fire.ly").WithStrictSerializer();

There are several other predefined options available:

* ``WithLegacySerializer`` - The default. Use the legacy parser, configured to be "lenient", allowing most errors that do not cause data loss. Note: these 
  parsers will not allow unknown values for enumerated elements or unknown elements. It can be further configured using the
  ``FhirClient.Settings.ParserSettings`` property. This is basically the default behaviour since SDK 1.
* ``WithStrictSerializer`` - Use the improved serializer, configured to parse the incoming data strictly.
* ``WithStrictLegacySerializer`` - Use the legacy serializer, configured to parse the incoming data strictly.
* ``WithLenientSerializer`` - Use the improved serializer, configured to ignore recoverable errors in the incoming data.
* ``WithPermissiveLegacySerializer`` - Use the legacy serializer, configured to be permissive.
* ``WithBackwardsCompatibleSerializer`` - Use the improved serializer, configured to ignore errors that can be caused when encountering data from a newer version of FHIR. NB: This may cause data loss.
* ``WithBackwardsCompatibleLegacySerializer`` - Use the legacy serializer, configured to ignore errors that can be caused when encountering data from a newer version of FHIR. NB: This may cause data loss.
* ``WithOstrichModeSerializer`` - Use the improved serializer, configured to ignore all errors. NB: This may cause data loss.
* ``WithOstrichModeLegacySerializer`` - Use the legacy serializer, configured to ignore all errors. NB: This may cause data loss.

The most obvious difference between the two families, in the context of the ``FhirClient`` will be the fact that
the new deserializer will throw a ``DeserializationFailed`` exception, while the legacy deserializer throws a ``FormatException``
(or subclass). For more info on the advantages of the new parsers, see the sections on :ref:`serialization <FHIR-parsing>`. 

These extension methods will allow you to easily configure the ``FhirClient`` to support common scenario's. If you need
even more control of the parsing and serialization behaviour of the client, you can implement the ``IFhirSerializationEngine`` interface, 
and update the ``Settings.SerializationEngine`` setting accordingly.

