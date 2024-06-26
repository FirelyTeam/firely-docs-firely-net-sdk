How to resolve artifacts from packages
---------------------------------------

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
