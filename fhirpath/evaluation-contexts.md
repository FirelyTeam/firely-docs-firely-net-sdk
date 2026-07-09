# Evaluation Contexts

The extension methods and the `CompiledExpression` all take an expression (as a string) and a second parameter, the `EvaluationContext`.
The context can normally be ignored, but is used to set specific environment-variables in case the defaults don't work out:

| Property | Purpose |
|----------|---------|
| `EvaluationContext.Resource` | Gets or sets the `PocoNode` returned by the `%resource` environment variable. Default is null. |
| `EvaluationContext.RootResource` | Gets or sets the `PocoNode` returned by the `%rootResource` environment variable. Default is null. |
| `EvaluationContext.Environment` | A dictionary of custom environment variables, made available in the expression under the `%` prefix followed by the variable name. |
| `EvaluationContext.Tracer` | A delegate that handles the output for the `trace()` function. |
| `FhirEvaluationContext.ElementResolver` | A delegate that resolves a uri to an instance of FHIR data (a `PocoNode`). This callback is used by the FHIR-specific method `resolve()`. |
| `FhirEvaluationContext.TerminologyService` | The terminology service used by the FHIR-specific method `memberOf()`. |

Note that `FhirEvaluationContext` (a subclass of `EvaluationContext`) is only used by the POCO extension methods for FhirPath, as it provides the properties for the FHIR-specific `resolve()` and `memberOf()` functions.
