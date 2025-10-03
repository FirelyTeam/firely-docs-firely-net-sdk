# Backbone Elements

Backbone elements are a key concept in FHIR, used to group related data within a resource. These elements are not standalone types but are instead complex structures embedded within a resource. In FHIR terminology, they are referred to as `BackboneElements`. For instance, the `Patient` resource includes a backbone element called `contact`.

```{image} ../images/fhir_patient_component.png
```

In the .NET SDK, a backbone element is implemented as a nested class within the corresponding resource class. The name of this subclass is derived from the backbone element's name, appended with the word `Component`. For example, the `contact` backbone element in the `Patient` resource is represented as a `ContactComponent` class within the `Patient` class:

```{image} ../images/sdk_patient_component.png
```

To add contact details to a `Patient` instance, you can create and configure a `ContactComponent` object as shown below:

```csharp
var contact = new Patient.ContactComponent();
contact.Name = new HumanName();
contact.Name.Family = "Parks";
// Configure additional contact details

pat.Contact.Add(contact);
```