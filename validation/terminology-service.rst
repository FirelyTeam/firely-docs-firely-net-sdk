.. _terminology-service:

=====================================
Using Terminology Services
=====================================

FHIR defines several operations related to terminologies. These operations let healthcare applications make use of codes 
and value sets without having to become experts in the fine details of code system, value set and concept map resources, and the underlying code systems and 
terminological principles. 

We have added support in the Firely .NET SDK to use these operations, both locally, as calling an external "Terminology Server" that have implemented these operations and 
return a result based on the input parameters. Both ``LocalTerminologyService`` as ``ExternalTerminologyService`` implement ``ITerminologyService``.

ITerminologyService
--------------------------
``ITerminologyService`` is an interface that describes all functions of a terminology service. ITerminologyService can for example be added to a FHIR validator to handle everything terminology related. 
All functions represent a FHIR Terminology operation and have a ``Parameters`` resource as input that represents the input parameters of the FHIR operation. There is a specific helper class for each operation to easily create this ``Parameters`` resource.
Each function returns the output of that specific operation, this can be a ``Parameters`` resource including all output parameters, or just a single ``Resource`` which was the result of your request.

The following functions are defined in the interface are:

- ``Task<Parameters> ValueSetValidateCode(Parameters parameters, string id = null, bool useGet = false)``

Represents the ``$validate-code`` operation on the ValueSet resource (https://www.hl7.org/fhir/valueset-operation-validate-code.html) and will validate that a coded value is in the set of codes allowed by a value set. 
The input parameters can be easily created using the ``ValidateCodeParameters`` class.

- ``Task<Parameters> CodeSystemValidateCode(Parameters parameters, string id = null, bool useGet = false)``

Represents the ``$validate-code`` operation on the CodeSystem resource (http://hl7.org/fhir/codesystem-operation-validate-code.html) and will validate that a coded value is in the code system.
The input parameters can be easily created using the ``ValidateCodeParameters`` class.

- ``Task<Resource> Expand(Parameters parameters, string id = null, bool useGet = false)``

Represents the ``$expand`` operation (http://hl7.org/fhir/valueset-operation-expand.html), which uses the definition of a value set to create a simple collection of codes suitable for use for data entry or validation.
The input parameters can be easily created using the ``ExpandParameters`` class.

- ``Task<Parameters> Lookup(Parameters parameters, bool useGet = false)``

Represents the ``$lookup`` operation (http://hl7.org/fhir/codesystem-operation-lookup.html), which given a given a code, system, or a Coding, get additional details about the concept, including definition, status, designations, and properties.
The input parameters can be easily created using the ``LookupParameters`` class.

- ``Task<Parameters> Translate(Parameters parameters, string id = null, bool useGet = false)``

Represents the ``translate`` operation ("http://hl7.org/fhir/conceptmap-operation-translate.html"), which translates a code from one value set to another, based on the existing value set and concept maps resources, and/or other additional knowledge of the processor.
The input parameters can be easily created using the ``TranslateParameters`` class.

- ``Task<Parameters> Subsumes(Parameters parameters, string id = null, bool useGet = false)``

Represents the ``subsumes`` operation ("http://hl7.org/fhir/codesystem-operation-subsumes.html") and tests the subsumption relationship between code/Coding A and code/Coding B given the semantics of subsumption in the underlying code system.
The input parameters can be easily created using the ``SubsumesParameters`` class.

- ``Task<Resource> Closure(Parameters parameters, bool useGet = false)``

Represents the ``closure`` operation ("http://hl7.org/fhir/conceptmap-operation-closure.html") and provides support for ongoing maintenance of a client-side transitive closure table based on server-side terminological logic. 
The input parameters can be easily created using the ``ClosureParameters`` class.

LocalTerminologyService
--------------------------

The Firely SDK support some terminology logic itself using the ``LocalTerminologyService`` meaning it doesn't call a third party to handle it, 
but does as much as it can internally.

.. note:: The ``LocalTerminologyService`` doesn't support all terminology features. To fully support all features a lot of terminology expertise is needed. So to make use of all advanced terminology features, use the ``ExternalTerminologyServer`` 
    to let a dedicated terminology server handle your requests.

The ``LocalTerminologyService`` requires an ``IResourceResolver`` to resolve all the FHIR resources needed to perform it's terminology logic. 

The following function is currently supported by the ``LocalTerminologyService``:

- ``Task<Parameters> ValueSetValidateCode(Parameters parameters, string id = null, bool useGet = false)``

This function validates that a coded value is in the set of codes allowed by a value set. The ValueSet is searched using the provided ``IResourceResolver``, for example an :ref:`package_source` containing all FHIR artifacts from a given package. 
Once the ValueSet is found, the provides code will be validates against that ValueSet. 

In some cases ``ValueSets`` are implicitly defined using the  ``valueSet`` element in a ``CodeSystem`` resource, which implicitly defines a ValueSet containing all codes from that CodeSystem.
If the ``IResourceResolver`` also implements ``IConformanceSource``, the ``LocalTerminologyService`` can still validate your code against such an implicitly defined ValueSet.


ExternalTerminologyService
--------------------------

``ExternalTerminologyService`` implements all functions of ``ITerminologyService`` and uses a ``FhirClient`` to connect to an external terminology server that will handle all operations.

All functions in the ``ExternalTerminologyService`` have a ``bool useGet = false`` parameter, which can be used to make the FHIR client use the ``GET`` rest operation, instead of ``POST`` when sending the request.

The example below first creates a ``FhirClient`` based on the FHIR endpoint of your favorite terminology server.  The ``ExternalTerminologyServer`` will use 
that client to connect to the terminology server and send a request to the server to expand a particular ``ValueSet`` using the ``$expand`` operation.
After processing your request, the terminology server returns the expanded ``ValueSet`` based on your input (or an ``OperationOutcome`` if something went wrong).

.. code-block:: csharp

    var client = new FhirClient("https://someterminologyserver.org/fhir");
    var svc = new ExternalTerminologyService(client);

    var parameters = new ExpandParameters()
        .WithValueSet(url: "http://snomed.info/sct?fhir_vs=refset/142321000036106")
        .WithFilter("met")
        .WithPaging(count: 10);

    var result = await svc.Expand(parameters) as ValueSet;


MultiTerminologyService
-----------------------

``MultiTerminologyService`` allows you to combine multiple terminology services. This is useful when you have terminology service that are specialized in certain ValueSet or if you want to first check if you can process codes locally before consulting an external terminology service.
The order of the terminology services added to the constructor decides which terminology service is consulted first. If a terminology service comes back with a result (true of false), the fallback services are not consulted anymore.

.. code-block:: csharp

    var local = new LocalTerminologyService(ZipSource.CreateValidationSource());
    
    var client = new FhirClient("https://someterminologyserver.org/fhir");
    var multi = new ExternalTerminologyService(client);

    var multiTermService = new MultiTerminologyService(local, multi);

In this example above, the local terminology service is always consulted first, but if the local service is indecisive (a ``FhirOperationException`` is thrown), the external service is consulted.

Routing
^^^^^^^^

Sometimes, you already know that certain ValueSets should be handled by a specific terminology service. 
For example, if you have a LocalTerminologyService with all your custom ValueSets, you already know that all other services will not be able to validate codes from those ValueSets.
That's when you want to introduce routing in your ``MultiTerminologyService``.

You can add routing information to the ``MultiTerminologyService`` by adding a ``TerminologyServiceRoutingSettings`` to the constructor.


.. code-block:: csharp

    var local = new LocalTerminologyService(ZipSource.CreateValidationSource());
    var localRouting = new TerminologyServiceRoutingSettings(local)
    {
        PreferredValueSets = new string[]{"http://fire.ly/ValueSet/*"}
    };

    var client = new FhirClient("https://someterminologyserver.org/fhir");
    var multi = new ExternalTerminologyService(client);
    var localRouting = new TerminologyServiceRoutingSettings(multi);
    {
        PreferredValueSets = new string[]{"http://hl7.fhir.org/ValueSet/*"}
    }

    var multiTermService = new MultiTerminologyService(localRouting, multiRouting);

.. note:: You can use a '*' to specify wildcards in the routing mechanism.

The example above will route all ValueSets starting with "http://fire.ly/ValueSet/" to the local terminology service first, and the ValueSets starting with "http://hl7.fhir.org/ValueSet/" to the external service.
All other incoming requests will be handles by the order the services have been passed to the constructor, in this case, first local, then external.




