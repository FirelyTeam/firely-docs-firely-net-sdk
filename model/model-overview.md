# Overview of the POCO model

The FHIR POCO (Plain Old CLR Object) model in the `Hl7.Fhir.Model` namespace maps FHIR resources and data types to idiomatic .NET classes. The UML diagram below shows the principal classes and interfaces and the relationships described in this chapter.

Historically, the class hierarchy has evolved: in FHIR STU3 and R4 most datatypes inherited directly from `Element`. R5 introduced intermediate types such as `Base`, `PrimitiveType`, and `DataType` to better structure shared behavior. The .NET SDK adopts these concepts retroactively, so the SDK POCOs for STU3 and R4 also use these base classes. As a result, the class hierarchy shown here reflects the SDK’s unified view and applies to all supported FHIR versions, even when the original STU3 and R4 definitions did not explicitly include these intermediate types.

The diagram explicitly includes `Code` and `Code<T>`: `Code<T>` is a SDK-specific construct (not part of the FHIR specification) that represents enumerable codes. The diagram also shows `PocoNode`; although this class is not part of the FHIR specification, it is important for integrating POCOs with code that uses `ITypedElement` and `ISourceNode`.

```mermaid
classDiagram
    class Base {
        <<abstract>>
        bool HasOverflow
        CompareChildren(...)
        TryGetValue(...)
        SetValue(...)
        EnumerateElements()
    }

    class IAnnotated {
        Annotations(Type)
    }

    class IAnnotatable {
        <<interface>>
        AddAnnotation(object)
        RemoveAnnotations(Type)
    }

    IAnnotatable --|> IAnnotated

    Base ..|> IAnnotatable

    class Element{
        <<abstract>>
        string ElementId
    }

    class PocoNode {
        PocoNode Parent
        string Name
        int Index
        IEnumerable~PocoNodeOrList~ Children()
        PocoNodeOrList Child(string name)
    }

    PocoNode ..|> ITypedElement
    PocoNode ..|> ISourceNode
    
    PocoNode *-- Base

    Element --|> Base
    
    BackboneElement --|> Element
    class BackboneElement {
        <<abstract>>
        List~Extension~ ModifierExtension
    }

    note for BackboneElement "Nested classes like Patient.Contact
     derive from BackboneElement"

    class DomainResource{
         <<abstract>>
        Narrative Text
        List~Resource~ Contained
    }

    note for DomainResource "Most resources derive
     from DomainResource"

    Resource --|> Base
    DomainResource --|> Resource

    class DataType {
        <<abstract>>
    }  

    PrimitiveType --|> DataType

    class PrimitiveType {
        <<abstract>>
        object JsonValue
        bool HasValidValue()
        bool TryConvertToSystemType(out Any)
    }

    note for PrimitiveType "All FHIR primitive types (e.g., FhirString,
    FhirBoolean) derive from PrimitiveType"

    Code --|> PrimitiveType

    class Code {
        string Value
        string Literal
        string System
    }

    CodeOfT --|> Code

    class CodeOfT {
        new T Value
    }

    note for DataType "Non-primitive datatypes
     derive from DataType"

    DataType --|> Element

    class Resource {
        <<abstract>>
        string ResourceType
        Uri ResourceBase
    }

    note for Resource "Bundle, Parameters, Binary
     derive directly from Resource"

```

## Dynamic types in the model
The SDK also provides dynamic types for representing instances that do not have a compiled POCO in the SDK. The parsers create these runtime types when they encounter unknown resource types or data that have no corresponding .NET class.

```mermaid
classDiagram

    class IDynamicType{
        <<interface>>
        string DynamicTypeName
    }

    DynamicResource --|> DomainResource
    DynamicResource ..|> IDynamicType
    DynamicPrimitive --|> PrimitiveType
    DynamicPrimitive ..|> IDynamicType
    DynamicDataType --|> DataType
    DynamicDataType ..|> IDynamicType
    DynamicInfraResource --|> Resource
    DynamicInfraResource ..|> IDynamicType
```

## Modifier extensions in the model
The FHIR specification treats modifier extensions differently from regular extensions: modifier extensions convey additional information that must not change the core meaning of the element they modify, and they therefore require careful handling. The SDK indicates which classes may carry modifier extensions by using the `IModifierExtendable` interface.

As shown in the diagram, most typical FHIR resources support both `Extension` and modifier extensions, but some infrastructure resources such as `Bundle` and `Parameters` (which derive directly from `Resource`) do not support extensions at all. Also, primitive types may have regular `Extension` instances but cannot have modifier extensions.

```mermaid
classDiagram
    class IModifierExtendable {
        <<interface>>
        List~Extension~ ModifierExtension
    }

    IModifierExtendable ..|> IExtendable

    BackboneElement ..|> IModifierExtendable
    BackboneType ..|> IModifierExtendable
    DomainResource ..|> IModifierExtendable
    DomainResource --|> Resource

    note for BackboneType "Non-primitive datatypes that can have
     modifier extensions derive from BackboneType"
    class BackboneType {
         <<abstract>>
    }

        class IExtendable {
        <<interface>>
        List~Extension~ Extension
    }


    BackboneType --|> DataType

    Element ..|> IExtendable
    Element --|> Base
    Resource --|> Base
    DataType --|> Element
``` 

