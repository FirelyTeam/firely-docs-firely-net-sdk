.. toctree::
   :maxdepth: 2
   :caption: Firely Products
   :hidden:

   Back to all Firely Products <https://docs.fire.ly>

.. toctree::
   :maxdepth: 3
   :caption: Firely .NET SDK
   :hidden:

   start
   model
   client
   parsing
   validation
   fhirpath
   cql
   resource-resolving
   contact
   releasenotes

.. list-table::
   :widths: 33 33 33
   :header-rows: 1

   * - STU3
     - R4
     - R5
   * -  .. image:: https://dev.azure.com/firely/firely-net-sdk/_apis/build/status/Continuous%20Build?branchName=develop-stu3
           :target: https://dev.azure.com/firely/firely-net-sdk/_build?definitionId=84&_a=summary&repositoryFilter=50&branchFilter=2405%2C2405%2C2405%2C2405%2C2405
     -  .. image:: https://dev.azure.com/firely/firely-net-sdk/_apis/build/status/Continuous%20Build?branchName=develop-r4
           :target: https://dev.azure.com/firely/firely-net-sdk/_build?definitionId=84&_a=summary&repositoryFilter=50&branchFilter=2407%2C2407%2C2407
     -  .. image:: https://dev.azure.com/firely/firely-net-sdk/_apis/build/status/FirelyTeam.firely-net-sdk?branchName=develop-r5
           :target: https://dev.azure.com/firely/firely-net-sdk/_build?definitionId=84&_a=summary&repositoryFilter=50&branchFilter=2373%2C2373
   * - .. image:: https://img.shields.io/nuget/dt/hl7.fhir.stu3.svg
           :target: https://www.nuget.org/packages/hl7.fhir.stu3
     -  .. image:: https://img.shields.io/nuget/dt/hl7.fhir.r4.svg
           :target: https://www.nuget.org/packages/hl7.fhir.r4
     - .. image:: https://img.shields.io/nuget/dt/hl7.fhir.r5.svg
           :target: https://www.nuget.org/packages/hl7.fhir.r5

`View our source code on Github <https://github.com/FirelyTeam/firely-net-sdk>`__

Welcome to the Firely .Net SDK's documentation!
===============================================

This is the documentation site for the support SDK for working with `HL7
FHIR <http://www.hl7.org/fhir>`__ on the Microsoft .NET platform.

.. important:: The old name of this product was FHIR .NET API. Since November 2020 we renamed it to **Firely .NET SDK**.
  It is still the same product from the same contributors only with another name.


The library provides:

- Class models for working with the FHIR data model using POCOs
- A REST client for working with FHIR-compliant servers
- Xml and Json parsers and serializers
- Helper classes to work with the specification metadata, and generation of differentials
- Validator to validate instances against profiles
- A lightweight in-memory terminology server

On these pages we provide you with the documentation you need to get up and running with the SDK.
We'll first explain how the FHIR model is represented in the SDK and give you code examples to work with the model.
The FhirClient and its methods will also be demonstrated. Within an hour you can create your own simple
FHIR client!

After those topics to get you started, we have added some pages that delve deeper into nice SDK features, such
as parsing and serializing FHIR data, working with transactions, and using the ResourceIdentity functionality.

.. note:: All code examples on these pages are for the STU3 version of the library. 

Please look at the :ref:`sdk-contact` page for ways to ask questions, contribute to the SDK, or reach out to other
.Net developers in the FHIR community.
