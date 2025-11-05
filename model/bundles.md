# Bundles

Bundles are a fundamental concept in FHIR, serving as containers for sets of resources. While `Bundle` is technically just another resource type, it has unique characteristics that make it special. Bundles are primarily used to facilitate the exchange of resource collections. For example, you will often encounter Bundles when performing a [search interaction](../client/search). In such cases, the server responds with a `Bundle`. Bundles are also used in other scenarios where FHIR requires the exchange of resource lists, as detailed in the [Bundle documentation](https://hl7.org/fhir//bundle.html#scope).

## Reading a Bundle

A `Bundle` includes metadata, such as its type and, in cases of [search](../client/search) or [history](../client/history) interactions, a `total` property indicating the number of results. The individual resources within a `Bundle` are stored in the `Bundle.Entry` property.

Each entry in a `Bundle` contains a `Entry.FullUrl` field, which holds the fully qualified URL identifying the resource. Since the SDK cannot determine the specific resource type in an entry, the `Entry.Resource` field is typed as the base class `Resource`. To access resource-specific fields, you need to cast the `Resource` to its actual type, similar to working with [choice elements](./choice-elements.md):

```csharp
foreach (var e in result.Entry)
{
        if (e is Patient p { Active: true })
        {
                        // Output the fully qualified URL of the resource:
                        Console.WriteLine("Full URL for this Patient resource: " + e.FullUrl);

                        // Example: Output the patient's last name from the first name element:
                        Console.WriteLine("Patient's last name: " + p.Name[0].Family);
        }
}
```

## Creating a Bundle

The properties you need to set when creating a `Bundle` depend on its [intended use case](https://hl7.org/fhir//bundle.html#using). For instance, if you are a server developer constructing a search result, you must include the `Bundle.total` property. Similarly, when creating a transaction, you need to specify the HTTP operation for each resource entry in the `Bundle`.

To add resources to a `Bundle`, you can populate the `Entry` list with `EntryComponent` instances. Each `EntryComponent` includes the resource and its fully qualified URL. Alternatively, you can use the `AddResourceEntry` method of the `Bundle` class, which simplifies the process when only the URL and resource are required. For more complex scenarios, manually creating `EntryComponent` instances may be more appropriate.

Here is an example demonstrating both approaches:

```csharp
var collection = new Bundle();
collection.Type = Bundle.BundleType.Collection;

// Adding the first entry manually
var first_entry = new Bundle.EntryComponent();
first_entry.FullUrl = $"{res1.ResourceBase}{res1.ResourceType}{res1.Id}";
first_entry.Resource = res1;
collection.Entry.Add(first_entry);

// Adding a second entry using AddResourceEntry
collection.AddResourceEntry(res2, "urn:uuid:01d04293-ed74-4f93-aa0a-2f096a693fb1");
```

In this example, we create a `Bundle` of type `Collection`, and for this kind of Bundle we need to make sure all resources have a `FullUrl`. The first resource, `res1`, already has a technical ID (`Resource.Id`). We construct its `FullUrl` using information from the resource instance, though the `ResourceIdentity` helper methods in the `Hl7.Fhir.Rest` namespace could also be used. For more details, see the [Resource Identity documentation](../client/resource-identity).

The second resource, `res2`, is new and does not yet have `Resource.Id`, so we generate a temporary one.

### Creating a Transaction Bundle

Constructing Bundles manually, as demonstrated earlier, can be tedious. To simplify this process, the SDK offers a `TransactionBuilder` class, which streamlines the creation of batch and transaction Bundles.

Creating a transaction Bundle involves three key steps:
1. Instantiate a `TransactionBuilder`, specifying the appropriate "base address" (typically the server's address you are interacting with) and the type of Bundle to create, such as `transaction`, `batch`, or [any other relevant Bundle type for batches](https://hl7.org/fhir//valueset-bundle-type.html#expansion).
2. Add one or more operations to the transaction by invoking the corresponding batch methods (`Update`, `Create`, `Delete`, or even `Search`).
3. Call the `ToBundle()` method to generate the final transaction Bundle.

The following example demonstrates how to create a batch transaction that atomically updates `res1` and creates a new resource `res2`:

```csharp
var tb = new TransactionBuilder("http://example.org/fhir", Bundle.BundleType.Batch);

tb.Update(res1);
tb.Create(res2);

var transactionBundle = tb.ToBundle();
```
