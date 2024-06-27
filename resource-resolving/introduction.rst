===========================
FHIR Conformance data
===========================

The SDK provides a mechanism to access the FHIR specification's `conformance data <https://hl7.org/fhir/conformance-module.html>`_. 
To put it simply, these are the metadata that describe the FHIR resources, profiles, valuesets, operations, and other parts 
of the FHIR specification and implementationm guides. FHIR's conformance data is itself represented as FHIR resources, and may be 
packaged in multiple formats, like a zip file (pre-R4) and NPM. Traditionally, the .NET SDK used its own `specification.zip` file to distribute
the conformance data initially, but now also supports using the NPM packages.

Each resource that is part of the FHIR specification (or an implementation guide) has a unique *canonical url*, that identifies that resource
uniquely. Conformance resources can refer to other conformance resources (e.g. profiles that refer to other profiles), and this is done using
the canonical url. The principal interface to get to conformance data is ``IResourceResolver``, which looks up resources by their canonical url.

The SDK supplies several implementations for this interface to access conformance data from different sources, like a filesystem, a zip or an NPM file.

