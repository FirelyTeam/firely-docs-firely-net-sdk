# Validation of FHIR data

There are two approaches to validation in the SDK:

* An in-memory validation of the POCO FHIR data model, that runs the most important structural checks (cardinalities, choice types, primitive formats, coded values). This is useful to
  do validation of the core FHIR resources and datatypes when working with POCO data.
* A *profile validator*, which can validate FHIR data against [profiles](http://hl7.org/fhir/profilelist.html) . These profiles contain the full gamut of FHIR validation rules, and
  are used to validate the data in the FHIR resources.
