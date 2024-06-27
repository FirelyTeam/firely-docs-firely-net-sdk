==================
Standard resolvers
==================

The SDK supplies a number of implementations for ``IResourceResolver``. These allow you to resolve resources from filesystem directories, 
zip files, but there are specific implementations that help with caching or combining several resolvers as well. Resolvers for which the name ends in `Source` also 
implement ``IConformanceSource``.

Below is an example of how to use a resolver to resolve a resource. The example uses several resolvers that are documented below:

.. code-block:: csharp

    var resolver = new CachedResolver(
        new MultiResolver(
            new DirectorySource("path/to/directory"),
            ZipSource.CreateValidationSource(),
            new WebResolver()));

    var resource = resolver.ResolveByUri("http://hl7.org/fhir/StructureDefinition/Patient");


DirectorySource
~~~~~~~~~~~~~~~
This source will scan a filesystem directory and will then be able to resolve the resources that are in files in that directory. If
the files contain Bundles, the resources in the bundle will be available as well. 

The directory can be scanned recursively or not, and files may be excluded using masks, using the constructor overloads that take a 
``DirectorySourceSettings`` object. 

There is also a ``CommonDirectorySource``. See below for more information.

ZipSource
~~~~~~~~~
This resolver will resolve resources from a zip file. It will unpack the zip file and then resolve the resources that are in the zip file
using a ``DirectorySource``. The location of extraction can be specified in the constructor. The source uses a timestamp-based approach to avoid \
unpacking the zip multiple times.

Previously, the SDK was shipped by default with a ``specification.zip`` file, which contained all the conformance resources of the core FHIR 
specification. To use this file, the ZipSource has a static factory method, ``CreateValidationSource`` that will create a ZipSource that refers 
this ``specification.zip``. As of version 4.0.0, the SDK no longer ships with a ``specification.zip`` file, and developers are encouraged to 
use the newer NPM files, that are more up-to-date and are now the main source of conformance resources for both the standard specification and 
implementation guides. The ``specification.zip`` files are no longer shipped with the SDK, but can be downloaded as separate NuGet packages,
named ``Hl7.Fhir.Specification.Data.R4``, etcetera.

There is also a ``CommonZipSource``. See below for more information.

WebResolver
~~~~~~~~~~~
This resolver will resolve resources from a web server. It will interpret the canonical uri as a URL, 
and will then download the resource from that URL. Note that this is not often useful, since conformance resources are usually supplied
using more performant methods, such as the ``ZipSource`` or NPM packages and there is no control over which conformance resources can be
downloaded and from which FHIR endpoints.

There is also a ``CommonWebResolver``. See below for more information.

'Common' flavours of the resolvers
**********************************
For the resolvers listed above, there is also a 'common' version (e.g. ``CommonDirectorySource``). These common versions are used when want
to write code against several versions of the FHIR specification. Whereas the 'normal' versions of the resolvers are tied to a specific version
of FHIR (and thus included in specific versions of the SDK), the 'common' versions are not tied to one version, and can be configured by 
passing a ``ModelInspector`` to the constructor. See :doc:`the section on ModelInspector</model/model-metadata>`.

MultiResolver
~~~~~~~~~~~~~
This resolver combines several "child" resolvers into one. When resolving a resource, it will try to resolve the resource from each of the supplied
resolvers in turn, until one of the resolvers can resolve the resource. The resolvers are tried in the order they are supplied to the constructor.

CachedResolver
~~~~~~~~~~~~~~
Most resolvers require I/O and resolution is therefore relatively slow. The ``CachedResolver`` is a decorator resolver that caches the resources
that are resolved so that they can be resolved faster the next time. The cache is an in-memory cache, and the cache can be cleared using the 
``Clear`` method. It supports several caching strategies, setting a cache duration and methods for invalidation by canonical uri.

.. note:: The cache in thread-safe, but the object returned are references from the cache, so if you modify the object, you will modify the object in the cache as well.

SnapshotSource
~~~~~~~~~~~~~~
This resolver is a decorator that will resolve resources using the underlying resolver, but if that resolver returns a ``StructureDefinition``,
it will make sure that a snapshot is available. If the snapshot is not available, this resolver will generate
a snapshot using the SDK's ``SnapshotGenerator``. It is possible to supply a pre-configured ``SnapshotGenerator`` to the constructor, otherwise a 
default generator is used.

InMemoryResourceResolver
~~~~~~~~~~~~~~~~~~~~~~~~
This resolver will resolve resources that are supplied to it in-memory. This is useful when you have resources that are not stored in files, for example
when resources are generated on-the-fly or for testing purposes.
