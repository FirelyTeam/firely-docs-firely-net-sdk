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
