
Accessing FHIR model metadata
-----------------------------
The classes representing the FHIR resources and datatypes are all generated from the metadata artifacts from the FHIR specification. More or less, each datatype and resource is a single class, and each element a property of those classes. Additional, the most important valuesets from the specification are converted to enumeration. Each of these carry
additional metadata expresses as attributes on the classes and properties. This is metadata like cardinality, allowed "datatype choices" for a polymorphic property etcetera. Here is a fragment of such a generated class:

.. code-block:: csharp

    [FhirType("Patient","http://hl7.org/fhir/StructureDefinition/Patient", IsResource=true)]
    public partial class Patient : Hl7.Fhir.Model.DomainResource
    {
        /// <summary>
        /// FHIR Type Name
        /// </summary>
        public override string TypeName { get { return "Patient"; } }

As you can see, this declaration expresses a few things:

* This is a C# class ``Patient``, which represents the FHIR resource ``Patient``.
* The URL for the definition of this resoure
* The fact that this is a ``DomainResource`` and hence a ``Resource``.

Additionally, each datatype has a ``TypeName`` property that contains the FHIR name for the type
(exactly the same value as in the ``FhirType`` attribute above it, but easier to access).

Properties are a bit more interesting:

.. code-block:: csharp

    /// <summary>
    /// Time of assessment
    /// </summary>
    [FhirElement("effective", InSummary=true, Order=150, Choice=ChoiceType.DatatypeChoice)]
    [CLSCompliant(false)]
    [AllowedTypes(typeof(Hl7.Fhir.Model.FhirDateTime),typeof(Hl7.Fhir.Model.Period))]
    [DataMember]
    public Hl7.Fhir.Model.DataType Effective
    {

This is a FHIR element called "effective" as we can see, and the metadata tells us that this element 
is defined to be present "in summary", a feature of FHIR to represent summarized search results.
As well, this is a *choice* property, which allows the property to take a FHIR ``dateTime`` or
``period`` type (encoded here as the implementing C# type). Additional attributes will specify the
cardinality of repeating elements and other details.

It is of course feasible to retrieve these attributes using the classes in the familiar ``System.Reflection`` namespace, but as the SDK itself frequently needs this data too, we have included a few heavily cached and optimized classes to easily retrieve the metadata present in these attributes. These are the ``ClassMapping``, the ``PropertyMapping`` and ``ModelInspector`` classes, which we will detail below.

Before we dive into those, however, it is important to understand how the different releases of FHIR (STU3, R5, etc.) are reflected in the generated assemblies. The FHIR SDK is packaged as a set of NuGet packages, one for each FHIR release, but there is a set of FHIR datatypes that are stable and practically the same across the releases. We call these classes the "common" classes, and these are packaged in a shared, common NuGet package. The common package contains classes like ``Resource``, ``HumanName`` and ``Identifier``, just to name a few. So, even if you are dealing with just one release, you will find the classes you need spread out over two assemblies: the "common" assembly and the assembly with the classes that represent FHIR datatypes and resources that are specific for a release, or at least vary too much from release to release to include in a shared assembly.

The ClassMapping
^^^^^^^^^^^^^^^^
ClassMapping holds much of the metadata for a given FHIR resource or datatype that is present on the ``FhirTypeAttribute`` shown above. In addition it lists the properties for the type, using a ``PropertyMapping``. You can get a (cached) ClassMapping for any POCO type by calling ``TryGetMappingForType()``, which will return ``false`` if the type has no FHIR metadata associated with it. The result is cached, so asking for the same mapping multiple times will return the same ClassMapping instance:

.. code-block:: csharp

    bool success = ClassMapping.TryGetMappingForType(typeof(Patient), FhirRelease.R4, out ClassMapping? mapping);

Note that you need to pass in a ``FhirRelease``, even if you are just working with a single version of the specification. Most of the time, therefore, it is easier to use the ``ModelInspector`` documented below, which will do the release-specific housekeeping for you.

The PropertyMapping
^^^^^^^^^^^^^^^^^^^
By far the most useful methods on the ``ClassMapping`` are those to find a specific property: there are three ways to get to the properties.

* Using the ``PropertyMappings`` property will get you a list of all the properties representing the elements of the FHIR datatype.
* ``FindMappedElementByName()`` finds the ``PropertyMapping`` passing it the exact name of the element.
* ``FindMappedElementByChoiceName()`` will do the former, but additionally enables you to find an element by its fully suffixed name when it is a choice element, e.g. passing ``onsetDateTime`` will return the PropertyMapping for the ``onset`` element.

The PropertyMapping contains all the metadata from the ``FhirElementAttribute``, including its element name, "in summary", order, just to name a few. Using its ``NativeProperty`` property, you can refer back to the underlying ``PropertyInfo`` if necessary.

The ModelInspector
^^^^^^^^^^^^^^^^^^
The SDK's ``ModelInspector`` is a set of ClassMappings for the datatypes and resources of *a single release of the FHIR specification*. It allows you to retrieve a ClassMapping by name, by .NET type or by it's "canonical", which is a unique URL assigned to that FHIR type by the FHIR specification (it normally looks like ``http://hl7.org/fhir/StructureDefinition/<typename>``). If you would be dealing with multiple releases of the specification, you would have one inspector for each of these releases.

You can create a ModelInspector yourself, and import types and mappings from assemblies by hand, but this is not necessary. You can either use the static ``ModelInfo.ModelInspector`` property or more explicitly call the static ``ModelInspector.ForAssembly()`` method, passing it a reference to the assembly with the POCO classes for a release of FHIR:

.. code-block:: csharp

    var inspector = ModelInfo.ModelInspector;
    // Equivalent: var inspector = ModelInspector.ForAssembly(typeof(ModelInfo).Assembly);   
    var someDatatypeMapping = inspector.FindClassMapping("HumanName");
    var aResourceMapping = inspector.FindClassMapping(typeof(Observation));
    var anotherResource = inspector.FindClassMappingByCanonical("http://hl7.org/fhir/StructureDefinition/Procedure");

As you can see, we are using a static reference to the ``ModelInfo`` class to get a reference to the model assembly. If you are working with :ref:`multiple versions <multiple-versions>`, you would use ``external alias`` to make each of these ``ModelInfo`` type references remain unique. Finally, you could use ``Assembly.Load()`` to dynamically load the necessary assemblies from storage, depending on which releases of FHIR you need to support.

It's perfectly fine to call ``ModelInfo.ModelInspector`` or ``ModelInspector.ForAssembly()`` repeatedly for the same assembly: it will return you the same ``ModelInspector`` instance.
