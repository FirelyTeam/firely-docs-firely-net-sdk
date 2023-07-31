==============
Using FhirPath
==============

The SDK contains a compiler and runtime for `FhirPath <http://hl7.org/fhirpath/>`_.
FhirPath is an extraction and navigation language and you can execute FhirPath expression both on :ref:`FHIR POCO classes <FHIR-model>`
and on :ref:`ITypedElement-based data <elementmodel-intro>`. For both, the following (extension) methods are available:

.. list-table:: Functions to evaluate FhirPath
   :widths: 10 90

   * - ``Select()``
     - Returns the nodes produced by the expression.
   * - ``Scalar()``
     - Returns a single scalar value produced by the expression.
   * - ``Predicate()``
     - Returns true if the expression evaluates to ``true`` or ``{}`` (empty) and ``false`` otherwise.
   * - ``IsTrue()``
     - Returns true if the expression evaluates to ``true`` and ``false`` otherwise.
   * - ``IsBoolean()``
     - Determines whether the expression evaluates to a given boolean value.

These functions exist as extension methods, so they can conveniently be called on either a POCO or ``ITypedElement``:

.. code-block:: csharp

   Patient p = new() {...}
   var hasName = p.IsTrue("Patient.name.exists()");

   ITypedElement ite = FhirXmlNode.Parse(...).ToTypedElement(...);
   hasName = ite.IsTrue("Patient.name.exists()");

The ``Select()`` method will return a list of nodes - this is often used to navigate within the object and select a subset of the nodes within it:

.. code-block:: csharp

   Patient p = new() {...}
   var extensions = p.Select("Patient.extension");

When you run the ``Select`` method on a POCO, the nodes you will get returned are POCO's, in this example they would be of type ``Hl7.Fhir.Model.Extension``.
When running FhirPath on ``ITypedElement``, the nodes will be of type ``ITypedElement`` (and its ``InstanceType`` in this case would be ``Extension``).

``Select()`` consistently returns a list of nodes, even on a non-repeating element of a primitive type, ``p.Select("Patient.active")``
would therefore return an ``IEnumerable<Base>``, for which the only member is a POCO of type ``FhirBoolean``, not a string (true/false).
It's worth noting that this function will still return .NET primitive values (as in .NET int32 etc) when these are encountered in the FhirPath expression, e.g.
``Select("4+5")``. If you want to avoid this, you can use the convenience method ``ToFhirValues()``, which will translate all such
.NET primitive values to their equivalent FHIR primitive types:

.. code-block:: csharp

   Patient p = new() {...}
   var t = p.Select("true").ToFhirValues().Single();
   Assert.IsTrue(t is FhirBoolean);
