The IResourceResolver interface
-------------------------------

The ``IResourceResolver`` interface is used to resolve conformance resources from the specification or an implementation guide
(e.g. StructureDefinition, ValueSet, CodeSystem). Newer implementations use the async version of this interface:

.. code-block:: csharp

     public interface IAsyncResourceResolver 
     {
        /// <summary>Find a resource based on its relative or absolute uri.</summary>
        Task<Resource> ResolveByUriAsync(string uri);

        /// <summary>Find a (conformance) resource based on its canonical uri.</summary>
        Task<Resource> ResolveByCanonicalUriAsync(string uri); // IConformanceResource
     }

Note that the principle method is ``ResolveByCanonicalUriAsync`` which resolves a canonical resource. The ``ResolveByUriAsync`` method is used 
by some implementations to resolve non-conformance resources, like example instances of resources but is otherwise not widely supported and normally
is just hardwired to use the ``ResolveByCanonicalUriAsync`` method. These methods will return the resolved resource or null if the resource is not
found.

There are a number of useful extension methods that can be used to simplify the use of the ``IAsyncResourceResolver`` interface:

* FindStructureDefinitionAsync(this IAsyncResourceResolver resolver, string uri) - resolves a StructureDefinition, 
* FindStructureDefinitionForCoreTypeAsync(this IAsyncResourceResolver resolver, string coreType) - resolves a StructureDefinition for a named FHIR resource or datatype.
* FindValueSetAsync(this IAsyncResourceResolver resolver, string uri) - resolves a ValueSet
* FindCodeSystemAsync(this IAsyncResourceResolver resolver, string uri) - resolves a CodeSystem

These methods will return the resolved resource or null if the resource is not found or if the resolved resource is not of the requested type.

There is a related interface ``IConformanceSource``, which extends ``IResourceResolver``. This interface has methods to find specific types of conformance 
resources using criteria other than the canonical uri:

.. code-block:: csharp

    public interface IConformanceSource : ICommonConformanceSource
    {
        /// <summary>
        /// Find a <see cref="CodeSystem"/> resource by a <see cref="ValueSet"/> canonical url that contains all codes from that codesystem.
        /// </summary>
        CodeSystem FindCodeSystemByValueSet(string valueSetUri);

        /// <summary>List all resource uris for the resources managed by the source, optionally filtered by type. (these are not Canonical Uris)</summary>
        IEnumerable<string> ListResourceUris(ResourceType? filter = null);

        /// <summary>Find <see cref="ConceptMap"/> resources which map from the given source to the given target.</summary>
        IEnumerable<ConceptMap> FindConceptMaps(string? sourceUri = null, string? targetUri = null);

        /// <summary>Finds a <see cref="NamingSystem"/> resource by matching any of a system's UniqueIds.</summary>
        NamingSystem? FindNamingSystem(string uniqueId);
    }

This interface is not supported by all implementations of IResourceResolver.
