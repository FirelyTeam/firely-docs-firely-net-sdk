.. _fhir-package-source:

-----------------------
The FHIR Package Source 
-----------------------

Since version 5 of the SDK (and version R4 of FHIR), the FHIR specification has introduced a new way to manage and distribute FHIR artifacts using 
NPM packages. This method of distribution enables third parties and standardization bodies a way to distribute their own FHIR artifacts for
FHIR implementation guides. Implementation guides usually depend on other implementation guides, and NPM's versioning and dependency management
provided a strong, and standard way to manage these dependencies.

Conceptually, a FHIR NPM Package is a collection of FHIR Resources (like a directory with a JSON file-per-resource), and the ``FhirPackageSource``
can resolve conformance resources (like ValueSets) from one or multiple of these packages. 

.. note:: This functionality can be found in a separate NuGet package called ``Firely.Fhir.Packages``. 
	
There are multiple ways to create a ``FhirPackageSource`` that queries for artifacts in packages.

.. note::  
    Since the Package library is not FHIR version aware, you always have to provide a ``ModelInspector`` to provide it with type information. 
    The easiest way to do this is by passing it the ``ModelInspector`` from the SDK's ``ModelInfo`` class.  This will pass the type information of the FHIR version you are currently using to the package source.  
    If you want to resolve artifacts from multiple versions of FHIR, you have to create a ``FhirPackageSource`` for each version.


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
	
