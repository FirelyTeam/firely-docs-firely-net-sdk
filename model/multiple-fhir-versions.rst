.. _multiple-versions:

===========================================
Using multiple FHIR versions in one project
===========================================

It could happen that in your .NET project you need FHIR version STU3 and R4 simultaneously. But how can you distinguish a STU3 ``Patient`` from a R4 ``Patient``?

The solution lies in creating aliases for the different projects. See the inclusion of the different Firely .NET SDK packages in the 
project file here:

.. code-block:: XML

   <Project Sdk="Microsoft.NET.Sdk">

   <PropertyGroup>
       <OutputType>Exe</OutputType>
       <TargetFramework>net8.0</TargetFramework>
   </PropertyGroup>

   <ItemGroup>
        <PackageReference Include="Hl7.Fhir.STU3" Version="5.5.0" Aliases="stu3" />ItemGroup>
        <PackageReference Include="Hl7.Fhir.R4" Version="5.5.0" Aliases="r4" />
        <PackageReference Include="Hl7.Fhir.R4B" Version="5.5.0" Aliases="r4b" />
        <PackageReference Include="Hl7.Fhir.R5" Version="5.5.0" Aliases="r5" />
   </ItemGroup>

   </Project>

To use the ``Patient`` for STU3 and the ``Patient`` for R4 in the same C# unit file, you must use the aliases 
we previously defined. Declaring the extern alias introduces an additional root-level namespace to the global namespace. 
So in your code you can this to expend the namespace, for example like this for a R4 ``Patient``:


.. code-block:: csharp

   extern alias r4;
   extern alias stu3;

   namespace MultipleVersions
   {
        // Somewhere inside a method:
        var patientR4 = new r4::Hl7.Fhir.Model.Patient();
        var patientSTU3 = new stu3::Hl7.Fhir.Model.Patient();
   }
 

To avoid having to repeat the namespace, you can use an alias in the ``using``:

.. code-block:: csharp

   using R4 = r4::Hl7.Fhir.Model;

    // and then later...
   var patient = new R4.Patient():

So all together it looks like this:

.. code-block:: csharp

    extern alias r4;
    extern alias stu3;

    using System;
    using Hl7.Fhir.Model;
    using R4 = r4::Hl7.Fhir.Model;
    using STU3 = stu3::Hl7.Fhir.Model;

    namespace MultipleVersions
    {
        class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Hello World!");

                var code = new Code();

                var patient1 = new STU3.Patient();
                var patient2 = new R4.Patient();

            }
        }
    }


Dealing with Specification.zip
==============================
Although the recommended way of working with FHIR metadata is using the :ref:`FhirPackageSource <package-source>`, the SDK originally depended on the ``specification.zip`` file. Since the different SDK versions all use the same physical file, it is not possible to use different versions of the SDK in one project when using the ``specification.zip`` file, unless we tweak our projects files:

.. code-block:: csharp

 <ItemGroup>
	<PackageReference Include="Hl7.Fhir.Specification.Data.STU3" Version="5.5.0" GeneratePathProperty="true" ExcludeAssets="contentFiles" />
	<PackageReference Include="Hl7.Fhir.Specification.Data.R4" Version="5.5.0" GeneratePathProperty="true" ExcludeAssets="contentFiles" />
 </ItemGroup>

 <ItemGroup>
	<Content Include="$(PkgHl7_Fhir_Specification_Data_STU3)\contentFiles\any\any\specification.zip">
		<Link>specification_STU3.zip</Link>
		<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
		<CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
		<Pack>false</Pack>
	</Content>
	<Content Include="$(PkgHl7_Fhir_Specification_Data_R4)\contentFiles\any\any\specification.zip">
		<Link>specification_R4_0.zip</Link>
		<CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
		<CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
		<Pack>false</Pack>
	</Content>
 </ItemGroup>

You will notice that the package reference uses the ``GeneratePathProperty`` to be able to "link" the different ``specification.zip`` to a unique name that includes the FHIR specification version. When building the project, the ``specification.zip`` files will be copied to the output directory with the new name, and should then also be referenced differently in code creating a resolver using the zip:

.. code-block:: csharp

	IResourceResolver zipSource = fhirVersion switch
	{
		FHIRVersion.N3_0 => 
			stu3::Hl7.Fhir.Specification.Source.ZipSource.CreateValidationSource(Path.Combine(CommonDirectorySource.SpecificationDirectory, "specification_STU3.zip")),
		FHIRVersion.N4_0 => 
			r4::Hl7.Fhir.Specification.Source.ZipSource.CreateValidationSource(Path.Combine(CommonDirectorySource.SpecificationDirectory, "specification_R4_0.zip")),
		_ => throw new NotSupportedException()
	}

More background and details can be found in `Brian's blog on multi-version FHIR <https://brianpos.com/2023/09/22/firely-sdk-and-multiple-fhir-versions>`_.









