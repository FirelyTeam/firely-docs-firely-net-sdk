
=====================================
Other (smaller) features of the POCOs
=====================================

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
