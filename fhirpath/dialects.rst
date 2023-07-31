Dialects of FhirPath
--------------------

FhirPath is most commonly used as an integrated language within HL7 FHIR. The language was (despite the name) designed to be used in other contexts than FHIR,
and is also used within `CQL <https://cql.hl7.org/index.html>`_ for example. Each of these contexts can add additional variables and functions to the basic FhirPath language.
FHIR itself defines its own extensions to the language in `an appendix to the FHIR specification <https://www.hl7.org/fhir/fhirpath.html>`_.

In the SDK, this distinction is visible. When you are executing a FhirPath expression against ``ITypedElement`` (which could represent all models, also those from CQL),
we are not assuming any context, and the expression can (by default) *only* use the basic FhirPath functions. This means for example that a Fhir-specific function
like ``resolve()`` is not available when executing FhirPath against ``ITypedElement``.  When you are using POCO's - which are specifically generated for FHIR, the SDK
will have support for these extra functions. So:

.. code-block:: csharp

     // FHIR specific functions are supported via the POCO extension methods
     Base fhirData = new FhirString("hello!");
     Assert.IsTrue(fhirData.IsTrue("hasValue()"));

     // FHIR specific functions do not work via the ITypedElement extension methods
     ITypedElement data = ElementNode.ForPrimitive("hello!");
     Assert.ThrowsException<ArgumentException>(() => data.IsTrue("hasValue()"));

It is possible to change this default behaviour for ``ITypedElement`` by installing the Fhir dialect before you first use one of the FhirPath evaluation functions.
To achieve this, you have to manipulate the default table of symbols used by the FhirPath compiler: ``FhirPathCompiler.DefaultSymbolTable.AddFhirExtensions();``
This is, however, a global setting, which might (or better: will) cause problems when different parts of your applications need to use different dialects.
To circumvent this problem, you will need to use the lower-level FhirPath support functions, as shown in the next section.

The following functions of the FHIR dialect are currently supported by the SDK:

.. list-table:: Supported functions of the FHIR dialect
   :widths: 10 90

   * - ``extension(url: string)``
     -  Will filter the input collection for items named "extension" with the given url. 
   * - ``hasValue()``
     - Returns true if the input collection contains a single value which is a FHIR primitive, and it has a primitive value.
   * - ``trace(name : string; selector : expression)``
     -  A selection expression that can be used to shape what is logged for the collection that is traced. 
   * - ``resolve()``
     - For each item in the collection locate the target of the reference, and add it to the resulting collection.
   * - ``ofType(type : identifier)``
     - Determines whether an element is of a specific type.    
   * - ``memberOf(valueset : string)``
     - Determines whether input is a member of a specific valueset.
   * - ``htmlChecks()``
     - When invoked on an xhtml element returns true if the rules around HTML usage are met, and false if they are not.
   * - ``lowBoundary : T``
     - This function returns the lowest possible value in the natural range expressed by the type it is invoked on.
   * - ``highBoundary : T``
     - This function returns the lowest possible value in the natural range expressed by the type it is invoked.
   * - ``comparable(quantity) : boolean``
     - This function returns true if the engine executing the FHIRPath statement can compare the singleton Quantity with the singleton other Quantity and determine their relationship to each other.