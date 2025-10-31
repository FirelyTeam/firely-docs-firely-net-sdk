# Transactions

In addition to individual CRUD operations, the `FhirClient` can submit transaction bundles to a FHIR server.
Use `TransactionAsync(Bundle transaction)` to send a transaction; the call requires a transaction `Bundle` that describes the set of operations the server should perform. The simplest way to construct such a bundle is with the `TransactionBuilder` helper.

## Using TransactionBuilder

`TransactionBuilder` provides a convenient API to build a transaction or batch `Bundle` without manually constructing the `Bundle` and its entries.
You can add interactions by calling the corresponding builder methods (for example `Create()`, `Update()`, `Read()`, `Delete()`). When you've added all entries, call `ToBundle()` to obtain the `Bundle` to pass to `TransactionAsync()`.

```csharp
var pat = new Patient() { /* set up data */ };
var client = new FhirClient("http://server.fire.ly");
var builder = new TransactionBuilder("http://server.fire.ly", Bundle.BundleType.Transaction); // or Batch

// Add a new entry to the bundle including the patient resource that needs to be created.
builder.Create(pat);

// Add a new entry to the bundle with instructions that Patient with id "1337" needs to be deleted.
builder.Delete("Patient", "1337");

// returns the transaction bundle
var bundle = builder.ToBundle();

// Send the transaction to the server.
var response = await client.TransactionAsync(bundle);
```

Below are the interactions you can add to a transaction bundle using `TransactionBuilder`:

- `Create()`: create a resource on the server.
- `Read()`: return a resource from the server.
- `Update()`: update a resource on the server.
- `Delete()`: delete a resource on the server.
- `Patch()`: patch a resource on the server.
- `Search()`: search for resources on the server using search criteria.
- `CapabilityStatement()`: request the server's `CapabilityStatement`.
- `ResourceHistory()`: request the historical versions of a specific resource.
- `CollectionHistory()`: request historical versions of all resources of a given type.
- `ServerHistory()`: request server-wide historical versions of resources.
- `Transaction()`: execute a nested transaction on the server.

## Conditional interactions

In addition to the standard interactions, `TransactionBuilder` supports the
conditional interactions defined by the FHIR specification. Several
builder methods accept an optional `SearchParams` parameter to express
the condition to use for the interaction. Example:

```csharp
var pat = new Patient() { /* set up data */ };
var client = new FhirClient("http://server.fire.ly");
var builder = new TransactionBuilder("http://server.fire.ly", Bundle.BundleType.Transaction);

// Update the patient resource with family name "Levin", and date of birth "01-01-1990"
var updateConditions = new SearchParams();
updateConditions.Add("birthdate", "1990-01-01");
updateConditions.Add("family", "Levin");
builder.Update(updateConditions, pat);

// Delete the patient with email address "test@example.org"
var deleteCondition = new SearchParams("email", "test@example.com");
builder.Delete("Patient", deleteCondition);

// returns the transaction bundle
var bundle = builder.ToBundle();

// Send the transaction to the server.
var response = await client.TransactionAsync(bundle);
```

The following builder methods support conditional interactions:

- `Create()`
- `Update()`
- `Delete()`
- `Patch()`
