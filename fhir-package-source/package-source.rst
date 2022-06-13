.. _packge_source:

FHIR Package Source
---------------------------

As of SDK version 4.0 you can use a FHIR Package source to resolve FHIR artifacts like ``StructureDefinitions``, ``ValueSets``, and ``CodeSystems`` from one ore multiple FHIR Package.
This can be from the FHIR core packages, but it can be from any FHIR Package you like. These for FHIR artifacts are needed for example for validating resources. Or to generate snapshots of profiles.

Initiate a FHIR Package Source 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There are multiple ways to create a ``FhirPackageSource`` instance, that include the packages you want to use.
If you have a local copy of the packages you want to use, you can just specify the package file paths to create your ``FhirPackageSource``.

.. code:: csharp

	var package1 = "c:/files/packages/package1.tgz";
	var package2 = "c:/files/packages/package2.tgz";
	FhirPackageSource resolver = new(new string[] { package1, package 2});

However, if you don't have the packages you want to resolve your artifacts from yet, you can specify a FHIR package server, and the names and versions (using an ``@``) of the packages to install the packages locally and create a ``FhirPackageSource`` immediately.

.. code:: csharp
	
	var package1 = "hl7.fhir.r3.core.xml@3.0.2";
	var package2 = "hl7.fhir.r3.expansions@3.0.2";
	FhirPackageSource resolver = new("https://packages.simplifier.net", new string[] { package1, package 2});

.. note:: You may need an active internet connection to use this. This function will first check if the specified packages are already installed locally before searching for the packages online.
	

How to resolve artifacts from packages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The most common use case for the FHIR Package source is to resolve conformance resources like ``StructureDefinitions``, ``ValueSets``, and ``CodeSystems``  from specific FHIR packages. 

All conformance resources can be resolved by specifying their canonical url.

.. code:: csharp

	FhirPackageSource resolver = new(new string[] { package1, package 2});
	var patientProfile = await resolver.ResolveByCanonicalUriAsync("http://hl7.org/fhir/StructureDefinition/Patient") as StructureDefinition;
	var genderValueSet = await resolver.ResolveByCanonicalUriAsync("http://hl7.org/fhir/ValueSet/administrative-gender") as ValueSet;

You can also find any resource by their type and id, or any file by their name or relative filepath.

.. code:: csharp

	FhirPackageSource resolver = new(new string[] { package1, package 2});

	//find resource by id
	StructureDefinition pat = await resolver.ResolveByUriAsync("StructureDefinition/Patient") as StructureDefinition;

	//find file by name
	Stream file = resolver.LoadArtifactByName("StructureDefinition-Patient.xml");

	//find file by path
	Stream file2 = resolver.LoadArtifactByPath("package/StructureDefinition-Patient.xml");

Next to that, there are some specific functions for certain resource types.

.. code:: csharp

	FhirPackageSource resolver = new(new string[] { package1, package 2});

	//Find CodeSystems by ValueSets
	CodeSystem cs = resolver.FindCodeSystemByValueSet("http://hl7.org/fhir/ValueSet/address-type");

	//Find ConceptMaps by source and/or target (sourceUri or targetUri can be null)
	ConceptMap cms = resolver.FindConceptMaps(sourceUri: "http://hl7.org/fhir/ValueSet/data-absent-reason", targetUri: "http://hl7.org/fhir/ValueSet/v3-NullFlavor");

	//Find NamingSystem by uniqueId
	NamingSystem ns = resolver.FindNamingSystem("http://snomed.info/sct");


Different FHIR Versions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is also a ``class`` in the SDK called ``CommonFhirPackageSource``. This ``class`` is located in the FHIR version agnostic part of the SDK, and this is actually where all the magic happens.
``FhirPackageSource`` is just a small layer on top of ``CommonFhirPackageSource``, and provides it with version information using a ``ModelInspector`` depending on which FHIR version of the SDK you are using.
In practice this means that you can't combine packages of different FHIR versions in a single ``FhirPackageSource``, because the operations will need to resolve to different FHIR models.



	