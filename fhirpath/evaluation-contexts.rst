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