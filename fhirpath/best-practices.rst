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

Set the TerminologyService in the FhirEvaluationContext
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To utilize the FhirPath function ``memberOf(valueset)``, you must define the ``TerminologyService`` property in the ``FhirEvaluationContext``. 
This is necessary to provide the FhirPath engine with a means to search for codes.

.. code-block:: csharp

   var ctx = FhirEvaluationContext.CreateDefault();
   ctx.TerminologyService = new LocalTerminologyService(resolver: ZipSource.CreateValidationSource());

   var result = new Code("male").Scalar("memberOf('http://hl7.org/fhir/ValueSet/administrative-gender')", context);