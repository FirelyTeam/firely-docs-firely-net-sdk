# Welcome to the Firely .NET SDK Documentation!

```{tip}
This is the documentation for the new SDK version 6.x. If you were familiar with version 5.x, we recommend checking out the section on what is new in SDK6 in the [release notes](releasenotes). The SDK5 documentation can still be found [here](https://docs.fire.ly/projects/Firely-NET-SDK/en/sdk5/)
```

This site provides comprehensive documentation for the Firely .NET SDK, a support SDK designed for working with [HL7 FHIR](http://www.hl7.org/fhir) on the Microsoft .NET platform.

The SDK offers the following features:

- Class models for working with the FHIR data model using POCOs.
- A REST client for interacting with FHIR-compliant servers.
- XML and JSON parsers and serializers.
- Helper classes for navigating StructureDefinitions and generating differentials.
- A lightweight, in-memory terminology service.
- Retrieval of FHIR metadata (profiles, value sets) from NPM packages.
- Validation of FHIR instances using profiles (also known as "profile validation").

HL7 has released multiple versions of FHIR (e.g. STU3, R4), and as a result, there are corresponding SDK versions for each FHIR release.

Each SDK version is tailored to its respective FHIR release, ensuring compatibility and support for the specific resources and data types of that release. Be sure to select the correct SDK version for the FHIR release you are working with—this is often determined by local regulations or national Implementation Guides.

It is also possible to write software that supports multiple FHIR specification releases by [using external aliases](model/multiple-fhir-versions).

Most of the SDK's code can be found in the [main repository](https://github.com/FirelyTeam/firely-net-sdk). However, the code for [working with packages](https://github.com/FirelyTeam/Firely.Fhir.Packages) and [profile validation](https://github.com/FirelyTeam/firely-validator-api) is maintained in separate repositories. The SDK includes generated code for the FHIR classes, which is created using a [custom-built code generator](https://github.com/microsoft/fhir-codegen).

```{note}
All code examples on these pages are for the R4 version of the library.
```

For more information on how to ask questions, contribute to the SDK, or connect with other .NET developers in the FHIR community, please visit the [](contact) page.
