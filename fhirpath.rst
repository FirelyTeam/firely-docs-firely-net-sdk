.. _fhirpath:

==============
Using FhirPath
==============

The SDK contains a compiler and runtime for `FhirPath <http://hl7.org/fhirpath/>`_.
FhirPath is an extraction and navigation language and you can execute FhirPath expression both on :ref:`FHIR POCO classes <FHIR-model>`
and on :ref:`ITypedElement-based data <elementmodel-intro>`. For both, the following (extension) methods are available:

.. list-table:: Functions to evaluate FhirPath
   :widths: 10 90

   * - ``Select()``
     - Returns the nodes produced by the expression.
   * - ``Scalar()``
     - Returns a single scalar value produced by the expression.
   * - ``Predicate()``
     - Returns true if the expression evaluates to ``true`` or ``{}`` (empty) and ``false`` otherwise.
   * - ``IsTrue()``
     - Returns true if the expression evaluates to ``true`` and ``false`` otherwise.
   * - ``IsBoolean()``
     - Determines whether the expression evaluates to a given boolean value.

These functions exist as extension methods, so they can conveniently be called on either a POCO or ``ITypedElement``:

.. code-block:: csharp

   Patient p = new() {...}
   var hasName = p.IsTrue("Patient.name.exists()");

   ITypedElement ite = FhirXmlNode.Parse(...).ToTypedElement(...);
   hasName = ite.IsTrue("Patient.name.exists()");

The ``Select()`` method will return a list of nodes - this is often used to navigate within the object and select a subset of the nodes within it:

.. code-block:: csharp

   Patient p = new() {...}
   var extensions = p.Select("Patient.extension");

When you run the ``Select`` method on a POCO, the nodes you will get returned are POCO's, in this example they would be of type ``Hl7.Fhir.Model.Extension``.
When running FhirPath on ``ITypedElement``, the nodes will be of type ``ITypedElement`` (and its ``InstanceType`` in this case would be ``Extension``).

``Select()`` consistently returns a list of nodes, even on a non-repeating element of a primitive type, ``p.Select("Patient.active")``
would therefore return an ``IEnumerable<Base>``, for which the only member is a POCO of type ``FhirBoolean``, not a string (true/false).
It's worth noting that this function will still return .NET primitive values (as in .NET int32 etc) when these are encountered in the FhirPath expression, e.g.
``Select("4+5")``. If you want to avoid this, you can use the convenience method ``ToFhirValues()``, which will translate all such
.NET primitive values to their equivalent FHIR primitive types:

.. code-block:: csharp

   Patient p = new() {...}
   var t = p.Select("true").ToFhirValues().Single();
   Assert.IsTrue(t is FhirBoolean);

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

Invoking the FhirPath Compiler directly
---------------------------------------
The FhirPath compiler is just another public class in the ``Hl7.FhirPath`` namespace. It has a constructor which takes an argument of type ``SymbolTable`` - the key
to full control over the installed dialect:

.. code-block:: csharp

  var symbolTable = new SymbolTable()
        .AddStandardFP()
        .AddFhirExtensions();
  var newCompiler = new FhirPathCompiler(symbolTable);

You can now use the compiler to:

* ``Compile()`` an expression to a ready-to-execute delegate (called ``CompiledExpression``)
* ``Parse()`` an expression to an abstract symbol tree, for display or debugging purposes

Invoking the ``CompiledExpression`` is equivalent to using the ``Select()`` function described above. The other functions, like ``IsBoolean``
are also available (as extension methods).

Evaluation Contexts
-------------------
The extension methods and the ``CompiledExpression`` all take an expression (as a string) and a second parameter, the ``EvaluationContext``.
The context can normally be ignored, but is used to set specific environment-variables in case the defaults don't work out:

.. list-table:: Properties in EvaluationContext
   :widths: 10 90

   * - ``EvaluationContext.Resource``
     - Gets or sets the node returned by the ``%resource`` environment variable. Default is null.
   * - ``EvaluationContext.RootResource``
     - Gets or sets the node returned by the ``%rootResource`` environment variable. Default is null.
   * - ``EvaluationContext.Tracer``
     - A delegate that handles the output for the ``trace()`` function.
   * - ``FhirEvaluationContext.ElementResolver``
     - A delegate that resolves an uri to an instance of FHIR data (``ITypedElement``). This callback is used by the FHIR specific method ``resolve()``.

Note that ``FhirEvaluationContext`` is only used by the POCO extension methods for FhirPath, as it provides a property for setting the resolver.

Best practices
--------------
Although it is seemingly easy to invoke FhirPath, there are a few details that are easy to get wrong.

Start evaluation from the root
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To make the ``resolve()`` function work well (e.g. to resolve to entries in a Bundle or to a contained resource), the FhirPath engine needs
to have "seen" all the resources while navigating through the data, which means you need to evaluate Bundles from their roots.

.. code-block:: csharp

  Bundle b = new() {...}

  // The engine has worked from the root of the bundle down, so it knows how to resolve to other entries
  var active = b.Select("Bundle.entry.ofType(Patient).organization.resolve()");

  // The engine was started from the nested Patient node, so does not know how to find other entries.
  var org = Bundle.entry.OfType<Patient>[0];
  var active2 = org.Select("organization.resolve()");

  // This is fine too, since the context is transferred from call to call.
  var org2 = b.Select("Bundle.entry.ofType(Patient)");
  var active3 = org2.Select("organization.resolve()");

Use a context constructor which takes a resource to set ``%resource``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Although not many FhirPath statements use the ``%resource`` and ``%rootResource`` environment variables, they *do* get used, and the default constructors will
make it easy for you to *not* set them (blame us for that). To make sure these variables work well, you should pass a sensible ``EvaluationContext`` to the
FhirPath functions, even though they are optional:

.. code-block:: csharp

   Patient p = new() {...}
   var hasName = p.IsTrue("Patient.name.exists()", new FhirEvaluationContext(p.ToScopedNode()));

As you can see, we are passing in a new ``FhirEvaluationContext``, constructed with a reference to the root of the object. Additionally, the FhirPath engine needs
its data to be a ``ScopedNode``. This is a wrapper for ``ITypedElement`` that keeps track of parent nodes, contained nodes
an entry nodes in a ``Bundle``, and does the heavy lifting for making ``resolve()`` work (see previous section).


Set the Resolver property in the FhirEvaluationContext
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Finally, the engine needs you to supply a delegate when you want ``resolve()`` to be able to reach out to instances of Resources (via uri) that it cannot locate itself.
The delegate you need to supply takes a single string parameter (the uri), and returns an ``ITypedElement``. Just like in the previous section, it would be best
if you call ``ToScopedNode()`` on it before you return the instance.


.. code-block:: csharp

   var ctx = new FhirEvaluationContext(p.ToScopedNode());
   ctx.Resolver = myResolver;

   ITypedElement myResolver(string uri)
   {
        var resolved = ...;
        return resolved.ToScopedNode();
   }

If you are thinking: couldn't this be easier? Yes, we think so - but most of the solutions would be breaking changes. We are working on it ;-)
