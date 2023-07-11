.. _package_source:

FHIR Package Source
---------------------------

You can use a FHIR Package source to resolve FHIR artifacts like ``StructureDefinitions``, ``ValueSets``, and ``CodeSystems`` from one or multiple FHIR NPM package.
These can be FHIR core packages or any other FHIR NPM packages. Such packages can be used in resource validation or snapshot generation, for example.


.. note:: As of SDK version 5.0, this functionality has been moved to a separate NuGet package called ``Firely.Fhir.Packages``. 

ModelInspector
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Since the Package library is not FHIR version aware, you always have to provide a ``ModelInspector`` to provide it with type information. The easiest way to do this is by passing it the ``ModelInspector`` from the SDK's ``ModelInfo`` class. 
This will pass the type information of the FHIR version you are currently using to the package source. 
If you want to resolve artifacts from multiple versions of FHIR, you have to create a ``FhirPackageSource`` for each version.

Initiate a FHIR Package Source 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
There are multiple ways to create a ``FhirPackageSource`` that queries for artifacts in packages.

If you have a local copy of the packages you want to use, you can just specify the package file paths to create your ``FhirPackageSource``.

.. code:: csharp

	var package1 = "c:/files/packages/package1.tgz";
	var package2 = "c:/files/packages/package2.tgz";
	FhirPackageSource resolver = new(ModelInfo.ModelInspector, new string[] { package1, package 2});

However, if you don't have the packages you want to resolve your artifacts from yet, you can specify a FHIR package server, and the names and versions (using an ``@``) of the packages to install the packages locally and create a ``FhirPackageSource`` immediately.

.. code:: csharp
	
	var package1 = "hl7.fhir.r3.core.xml@3.0.2";
	var package2 = "hl7.fhir.r3.expansions@3.0.2";
	FhirPackageSource resolver = new(ModelInfo.ModelInspector, "https://packages.simplifier.net", new string[] { package1, package 2});

In case you just want to resolve FHIR Core artifacts, you can create a ``FhirPackageSource`` by calling the ``CreateCorePackage()`` method, which will pull the core FHIR packages of the version specified from a package server.

.. code:: csharp

	var packageServer = "https://packages.simplifier.net";
	var fhirRelease = FhirRelease.STU3;
	FhirPackageSource resolver = FhirPackageSource.CreateCorePackageSource(ModelInfo.ModelInspector, fhirRelease, packageServerUrl);


.. note:: You need an active internet connection to connect to  a package server. The FhirPackageSource will first check if the specified packages are already installed locally before searching for the packages online. FHIR packages are installed by default in ``C:\Users\{user}\.fhir\packages\`` (Windows), ``~/.fhir/packages/`` (macOS), or ``~/.fhir/packages/`` (Linux).
	

How to resolve artifacts from packages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The most common use case for the FHIR Package source is to resolve conformance resources like ``StructureDefinitions``, ``ValueSets``, and ``CodeSystems``  from specific FHIR packages. 
``FhirPackageSource`` implements ``IAsyncResourceResolver``, ``IArtifactSource``, and ``IConformanceSource``.
That means all conformance resources can be resolved by specifying their canonical url.

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

