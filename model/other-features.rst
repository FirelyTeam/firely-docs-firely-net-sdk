=====================================
Other (smaller) features of the POCOs
=====================================

Fluent initializers
--------------------
For several data types, the SDK provides you with extra initialization methods.
Visual Studio's IntelliSense will help you to view the possibilities while you type, or you can take
a look at ``Hl7.Fhir.Model`` with the Object Browser to view the methods, plus their attributes as well.

For the ``HumanName`` data type, the SDK has added some methods to make it easier to construct a
name in one go, using fluent notation:

.. code-block:: csharp

	pat.Name.Add(new HumanName().WithGiven("Christopher").WithGiven("C.H.").AndFamily("Parks"));

If you need to fill in more than the ``Given`` and ``Family`` fields, you could first construct
a ``HumanName`` instance in this manner, and add to the fields later on. Or you could choose not
to use this notation, but instead fill in all the fields the way it was explained in the other
paragraphs.



The ``IIdentifiable<T>`` interface
----------------------------------
All FHIR resources that have an element called ```Identifier`` will have a POCO that implements ``IIdentifiable<T>``, where ``T`` is the type of the ``Identifier`` property.
In practice there are only two types used for ``T``, which means that there is a ``IIdentifiable<Identifier>`` and ``IIdentifiable<List<Identifier>>``. ``IIdentifiable<T>`` 
derives from the empty marker interface ``IIdentifiable``. There are extension methods ``GetIdentifier()`` and ``TryGetIdentifier()`` available to quickly get the value 
of an identifier by system, given a POCO that implements ``IIdentifiable``.

The ``ICode<T>`` interface
--------------------------
The binding of the FHIR model for CQL defines which elements are the "primary code" element for a resource. Often, these are elements called "Code", but designers of the
POCO's have sometimes chosen a different name for the element that codifies a POCO. We have used the binding for CQL to identify each such element. A resource that has
a primary code will implement ``ICode<T>`` to get and set that code. Currently, ``T`` can be:
 
* ``Code``
* ``CodeableConcept``
* ``List<CodeableConcept>``
* ``FhirString``
* ``DataType``

The ``DataType`` (which is abstract) is used when the coded property is a choice (e.g. many coded elements related to Medication are defined to be Coding or Reference). 
Although this set is currently limited, the CQL binding might change in the future, so implementers are advised to accomodate for all coded types (code, Coding, CodeableConcept, Quantity), or the data types (string, uri))
to appear here.

``ICode<T>`` derives from the ``ICode`` interface, which has a single method ``ToCodings()`` that returns the coded contents in the form of a list of ``Coding``. Coded 
types are converted to ``Coding`` as described in https://hl7.org/fhir/terminologies.html#4.1.
