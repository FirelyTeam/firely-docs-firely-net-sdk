===========================
FHIR Package Source
===========================

Many people arrange their use-cases into projects (described by implementation guides). As these projects build upon (depend on) other projects, and these is a need to share and distribute projects, a management system became necessary.
This management system is called "FHIR NPM packages". A FHIR NPM Package is a collection of FHIR Resources (like a directory with a JSON file-per-resource).

You can use a FHIR Package source to resolve these resource like ``StructureDefinitions``, ``ValueSets``, and ``CodeSystems`` from one or multiple FHIR NPM packages.
These can be FHIR core packages or any other FHIR NPM packages. Such packages can be used in resource validation or snapshot generation, for example.

.. note:: As of SDK version 5.0, this functionality has been moved to a separate NuGet package called ``Firely.Fhir.Packages``. 