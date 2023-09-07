================
Working with CQL
================

As a result of the cooperation between Firely and `NCQA <https://www.ncqa.org/>`_, the SDK contains a full CQL Engine
and a tool to convert ELM-based CQL logic to C#.

To start working with CQL, you need to do two things:

- Install the CQL Packager .NET tool, which will allow you to read ELM files, turn them into C# and compile them. This is done by running ``dotnet tool install Hl7.Cql.Packager --global`` (add ``--prerelease`` if you are feeling adventurous).
- Add a reference to the `main CQL package <https://www.nuget.org/packages/Hl7.Cql>`_ to your project.

The packager tool will generate the C# that the engine can run using the engine. For example, if we had compiled a library called ``BCSEHEDISMY2022`` (version 1.0.0), the packager will create a class called ``BCSEHEDISMY2022_1_0_0`` that can be invoked like this:

.. code-block:: csharp

	using Hl7.Cql.Fhir;
	using Hl7.Cql.Packaging;
	using Hl7.Cql.Primitives;
	using Hl7.Fhir.Model;

	// Set up the parameters for the CQL library
	private readonly IDictionary<string, object> MY2023 =
	  new Dictionary<string, object>
	  {
		{
		  "Measurement Period",
			new CqlInterval<CqlDateTime>(
			new CqlDateTime(2023, 1, 1, 0, 0, 0, 0, 0, 0),
			new CqlDateTime(2023, 12, 31, 0, 0, 0, 0, 0, 0),
			true,
			true)
		}
	  };

	var patientEverything = new Bundle();  // add some data
	var context = FhirCqlContext.ForBundle(patientEverything, MY2023);
	var bcs = new BCSEHEDISMY2022_1_0_0(context); // instantiate the library
	var numerator = bcs.Numerator(); // invoke one of the definition in the library

More information can be found in the `GitHub repository for the CQL Engine <https://github.com/FirelyTeam/firely-cql-sdk#getting-started>`_.