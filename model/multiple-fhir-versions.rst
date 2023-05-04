.. _multiple-versions:

Using multiple FHIR versions in one project
-------------------------------------------

It could happen that in your .NET project you need FHIR version STU3 and R4 simultaneously. But how can you distinguish a STU3 ``Patient`` from a R4 ``Patient``?

The solution lies in creating aliases for the different projects. See the inclusion of the different Firely .NET SDK packages in the 
project file here:

.. code-block:: XML

   <Project Sdk="Microsoft.NET.Sdk">

   <PropertyGroup>
       <OutputType>Exe</OutputType>
       <TargetFramework>netcoreapp3.1</TargetFramework>
   </PropertyGroup>

   <ItemGroup>
       <PackageReference Include="Hl7.Fhir.STU3" Version="3.0.0" />
       <PackageReference Include="Hl7.Fhir.R4" Version="3.0.0" />
   </ItemGroup>

   <Target Name="AddPackageAliases" BeforeTargets="ResolveReferences" Outputs="%(PackageReference.Identity)">
       <ItemGroup>
       <ReferencePath Condition="'%(FileName)'=='Hl7.Fhir.STU3.Core'">
           <Aliases>stu3</Aliases>
       </ReferencePath>
       <ReferencePath Condition="'%(FileName)'=='Hl7.Fhir.R4.Core'">
           <Aliases>r4</Aliases>
       </ReferencePath>
       </ItemGroup>
   </Target>

   </Project>

I created here an alias for the STU3 (named ``stu3``) and for the R4 (named ``r4``).

To use the ``Patient`` for STU3 and the ``Patient`` for R4 in the same C# unit file, you must use the aliases 
we previously defined:

.. code-block:: csharp

   extern alias r4;
   extern alias stu3;

   namespace MultipleVersions
   {
    // code here
   }

Declaring the extern alias introduces an additional root-level namespace to the global namespace. 
So in your code you can this to expend the namespace, for example like this for a R4 ``Patient``:

.. code-block:: csharp

    var patient = new  r4::Hl7.Fhir.Model.Patient();

When you don't want to fully express the namespace everytime, you can use an alias 
in the using part:

.. code-block:: csharp

   using R4 = r4::Hl7.Fhir.Model;

  /*  Inside class: */

   var patient = new R4.Patient():

So all together it looks like this:

.. code-block:: csharp

    extern alias r4;
    extern alias stu3;

    using Hl7.Fhir.Model;
    using System;
    using R4 = r4::Hl7.Fhir.Model;
    using Stu3 = stu3::Hl7.Fhir.Model;

    namespace MultipleVersions
    {
        class Program
        {
            static void Main(string[] args)
            {
                Console.WriteLine("Hello World!");

                var code = new Code();

                var patient1 = new Stu3.Patient();

                var patient2 = new R4.Patient();

            }
        }
    }
