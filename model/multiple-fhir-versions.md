# Using multiple FHIR versions in one project

In a .NET project you may need to work with multiple FHIR versions at the same time (for example, `STU3` and `R4`). This raises the question: how do you distinguish a `Patient` from `STU3` and a `Patient` from `R4`?

The recommended approach is to add assembly aliases to the different Firely .NET SDK packages and then use `extern alias` in your C# files. The following `csproj` snippet shows how to assign aliases to each FHIR package:

```xml
<Project Sdk="Microsoft.NET.Sdk">

<PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net8.0</TargetFramework>
</PropertyGroup>

<ItemGroup>
         <PackageReference Include="Hl7.Fhir.STU3" Version="6.0.1" Aliases="stu3" />
         <PackageReference Include="Hl7.Fhir.R4" Version="6.0.1" Aliases="r4" />
         <PackageReference Include="Hl7.Fhir.R4B" Version="6.0.1" Aliases="r4b" />
         <PackageReference Include="Hl7.Fhir.R5" Version="6.0.1" Aliases="r5" />
</ItemGroup>

</Project>
```

To use both the `STU3` and `R4` versions of `Patient` in the same C# file, declare the `extern alias` identifiers at the top of the file. This creates a new root-level namespace for each aliased assembly, which lets you refer to types from each version explicitly:

```csharp
extern alias r4;
extern alias stu3;

namespace MultipleVersions
{
         // Somewhere inside a method:
         var patientR4 = new r4::Hl7.Fhir.Model.Patient();
         var patientSTU3 = new stu3::Hl7.Fhir.Model.Patient();
}
```

To avoid repeating the fully qualified names, you can introduce a `using` alias for the model namespace of each version:

```csharp
using R4 = r4::Hl7.Fhir.Model;

 // and then later...
var patient = new R4.Patient():
```

Putting it all together:

```csharp
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
```

## Dealing with Specification.zip

Although the preferred way to work with FHIR metadata in the SDK is to use the `FhirPackageSource`, older versions relied on a `specification.zip` file. Because different SDK packages can reference the same physical `specification.zip`, using those packages together can cause conflicts. You can avoid this by giving each referenced `specification.zip` a unique name at build time:

```xml
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
```

The `GeneratePathProperty` property exposes the package path so you can link each `specification.zip` under a unique file name that includes the FHIR version. When the project is built, these files are copied to the output folder with their new names; you can then load the appropriate zip file when creating a validation resolver:

```csharp
IResourceResolver zipSource = fhirVersion switch
{
                FHIRVersion.N3_0 =>
                                stu3::Hl7.Fhir.Specification.Source.ZipSource.CreateValidationSource(Path.Combine(CommonDirectorySource.SpecificationDirectory, "specification_STU3.zip")),
                FHIRVersion.N4_0 =>
                                r4::Hl7.Fhir.Specification.Source.ZipSource.CreateValidationSource(Path.Combine(CommonDirectorySource.SpecificationDirectory, "specification_R4_0.zip")),
                _ => throw new NotSupportedException()
}
```

For more information, see [Brian Postlethwaite's blog post on using multiple FHIR versions with the Firely SDK](https://brianpos.com/2023/09/22/firely-sdk-and-multiple-fhir-versions).
