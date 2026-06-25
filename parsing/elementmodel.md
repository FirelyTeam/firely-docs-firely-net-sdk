# ElementModel (legacy)

```{note}
The ElementModel API described in this section predates the current serialization stack introduced in SDK 5 and 6. It is still functional and shipped with the SDK, but is no longer the recommended approach for most use cases. This documentation is kept for users who are maintaining code written against the older API and want a gradual upgrade path.

For current serialization and deserialization, see {doc}`deserialization` and {doc}`serialization`.
```

The ElementModel is a low-level, version-independent in-memory representation of FHIR data as a tree of nodes. Unlike the POCO classes, it is not tied to a specific FHIR release and can represent data that does not conform to any known profile. The POCO parsers and serializers are built on top of this abstraction.

```{toctree}
:maxdepth: 2
:hidden: true

intro-to-elementmodel
isourcenode
itypedelement
itypedelement-serialization
elementmodel-overview
itypedelement-custom
errorhandling
```
