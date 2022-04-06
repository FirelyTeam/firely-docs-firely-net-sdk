.. _validation:

=========================
Validation of FHIR data
=========================

There are two approaches to validation in the SDK:

* A validation based on ``System.ComponentModel.DataAnnotations``, that runs the most important validation checks on the POCO FHIR datamodel in memory. This is useful to
  do validation of the core FHIR resources and datatypes when working with POCO data.
* A *profile validator*, which can validate FHIR data against `profiles <http://hl7.org/fhir/profilelist.html>`_ . This validator uses ``ITypedElement`` as input
  for instance data, and any number of `StructureDefinitions` expressing profiles to validate against. See :ref:`elementmodel-intro` for more information on ``ITypedElement``.

.. toctree::
   :maxdepth: 3
   :hidden:

   validation/poco-validation
   validation/profile-validation


