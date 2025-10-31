# CRUD interactions

A `FhirClient` instance named `client` was created in the previous topic. Below we use that client to perform the common CRUD operations against a FHIR server.

## Create a new resource

To create and store a new resource on the server, use `CreateAsync`:

```csharp
var patient = new Patient { /* set up data */ };
var createdPatient = await client.CreateAsync<Patient>(patient);
```

On failure this call typically throws a `FhirOperationException`. That exception exposes an `Outcome` property containing an [OperationOutcome] resource with machine- and human-readable details about the error.

On success the server returns the stored representation of the resource, including its id and metadata. The server may also modify or populate elements (for example by adding defaults), so the returned instance can differ from what you sent.

## Reading an existing resource

To read a resource you need its resource id or a full resource URL. The client provides overloads that accept either a relative string (for example `"Patient/31"`) or an absolute `Uri` (as long as it points to the same endpoint used when creating the `FhirClient`). Examples:

```csharp
// Read the current version of a Patient resource with technical id '31'
var location_A = new Uri("http://server.fire.ly/Patient/31");
var pat_A = await client.ReadAsync<Patient>(location_A);
// or
var pat_A = await client.ReadAsync<Patient>("Patient/31");

// Read a specific version of a Patient resource with technical id '32' and version id '4'
var location_B = new Uri("http://server.fire.ly/Patient/32/_history/4");
var pat_B = await client.ReadAsync<Patient>(location_B);
// or
var pat_B = await client.ReadAsync<Patient>("Patient/32/_history/4");
```

```{tip}
The SDK provides a `ResourceIdentity` class to help construct resource URIs. Since `ReadAsync` only takes URLs as parameters, if you have the resource type and its ID as distinct data variables, use `ResourceIdentity`:

```csharp
var patResultC = await client.ReadAsync<Patient>(ResourceIdentity.Build("Patient","33"));
```

Note that `ReadAsync` can be used to fetch the most recent version of a resource as well as a specific version, and thus covers the two logical REST interactions `read` and `vread`.

## Updating a resource

After you retrieve a resource, edit its contents and send it back to the server using `UpdateAsync`. The call takes the resource instance previously retrieved as its parameter:

```csharp
// Add a name to the patient, and update
pat_A.Name.Add(new HumanName().WithGiven("Christopher").AndFamily("Brown"));
var updated_pat = await client.UpdateAsync<Patient>(pat_A);
```

There is always a chance that someone else updated the resource between your read and your update. Servers that are version-aware may refuse your update and return HTTP 409 (Conflict), which causes `UpdateAsync` to throw a `FhirOperationException` with the same status code. Clients that want to request version-aware behavior can pass the optional second parameter `versionAware` set to `true`. This will make the client perform the update conditionally based on the resource version.

## Deleting a resource

The `DeleteAsync` interaction on the `FhirClient` deletes a resource on the server. It is up to the server to decide whether the resource is permanently removed or whether previous versions remain available. `DeleteAsync` has multiple overloads to allow deletion by URL, by resource type and id, or by passing an existing resource instance:

```csharp
// Delete based on a url or resource location
var location = new Uri("http://server.fire.ly/Patient/33");
await client.DeleteAsync(location);
// or
await client.DeleteAsync("Patient/33");

// You may also delete based on an existing resource instance
await client.DeleteAsync(pat_A);
```

`DeleteAsync` will throw a `FhirOperationException` if the resource was already deleted, did not exist, or the server returned an error indicating a failure.

Note that sending an update to a resource after it has been deleted is not necessarily an error and may effectively "undelete" it, depending on the server's behavior.

## Patching resources

The `PatchAsync` interaction lets you send a partial update to a resource on the server without sending the entire resource. The SDK supports the `FHIRPath Patch` format, which defines a set of operations to apply to a resource using a `Parameters` resource in the request body. Full documentation is available in the [FHIR specification for FhirPatch](http://hl7.org/fhir/fhirpatch.html).

To use `PatchAsync`, create a `Parameters` resource containing the patch operations you want to apply. You can build this `Parameters` resource either by adding `Parameter` entries manually:

```csharp
var patch = new Parameters();
patch.Parameter.Add(new Parameters.ParameterComponent()
{
        Name = "OPERATION",
        Part = new List<Parameters.ParameterComponent>()
        {
                new Parameters.ParameterComponent()
                {
                        Name = "type",
                        Value = new FhirString("replace")
                },
                new Parameters.ParameterComponent()
                {
                        Name = "path",
                        Value = new FhirString("Patient.name[0].given[0]")
                },
                new Parameters.ParameterComponent()
                {
                        Name = "value",
                        Value = new FhirString("John")
                }
        }
});
```

or by using one of the helper extension methods provided by the SDK:

```csharp
var patch = new Parameters();
patch.AddReplacePatchOperation("Patient.name[0].given[0]", new FhirString("John"));
```

Both approaches result in the same patch: replacing the first given name of the first `name` element of the `Patient` with "John".

Other helper methods available include `AddAddPatchOperation`, `AddInsertPatchOperation`, `AddMovePatchOperation`, and `AddDeletePatchOperation`, depending on the patch operation you need.

After building the `Parameters` resource, call `PatchAsync`:

```csharp
var patientId = "Patient/1";
var patched_pat = await client.PatchAsync<Patient>(patientId, patch);
```

If the patch succeeds, `PatchAsync` returns the patched resource. On failure it throws a `FhirOperationException` with details.

## Conditional interactions

The SDK supports conditional versions of `CreateAsync`, `UpdateAsync`, `DeleteAsync`, and `PatchAsync`. Not all servers support conditional interactions; a server may return HTTP 412 with an [OperationOutcome] to indicate lack of support.

Conditional interactions use search parameters to identify target resources. Check the resource type page in the HL7 FHIR specification (http://www.hl7.org/fhir/resourcelist.html) for available search parameters for the resource type you work with. Construct the conditions using `SearchParams`:

```csharp
var conditions = new SearchParams();
conditions.Add("identifier", "http://ids.acme.org|123456");
```

```{tip}
See [Searching for resources](searching) for more explanation about `SearchParams` and example search syntax.
```

For `CreateAsync`, the server can check whether an equivalent resource already exists based on the provided search parameters:

```csharp
var created_pat_A = await client.CreateAsync<Patient>(pat, conditions);
```

If no matches are found, the server will create the resource. If exactly one match is found, the server will not create a duplicate and will return HTTP 200 (OK). In either case, `created_pat_A` contains the resource returned by the server, unless you configured the `FhirClient` to request a minimal representation. If multiple resources match, the server returns an error.

Conditional `UpdateAsync` works similarly: provide a `SearchParams` object to identify the target:

```csharp
// using the same conditions as in the previous example
var updated_pat_A = await client.UpdateAsync<Patient>(pat, conditions);
```

If a single match is found, the update is applied to that resource. If no matches are found, the server treats the request as a create. Multiple matches produce an error.

Conditional `PatchAsync` behaves like `UpdateAsync` but takes a `Parameters` resource that describes the patch operations:

```csharp
// using the same conditions as in the previous example
// and a patch parameters defined in the patch section
var patched_pat_A = await client.PatchAsync<Patient>(conditions, patchparams);
```

If a single match is found, the patch is applied. If no match is found, the server will return `404 Not Found`. If multiple matches are found, the server will return `412 Precondition Failed`.

The conditional `DeleteAsync` takes the resource type as the first argument and the search parameters as the second:

```csharp
await client.DeleteAsync("Patient", conditions);
```

If no match is found the server will return an error. If one match is found that resource will be deleted. If multiple instances match, the server may choose to delete all matched resources or return an error.

## Refreshing data

If you have held a resource for some time, its data may have changed on the server. You can refresh your local copy at any time by calling `RefreshAsync`, passing the resource instance returned by a previous `ReadAsync`, `CreateAsync`, or `UpdateAsync`:

```csharp
var refreshed_pat = await client.RefreshAsync<Patient>(pat_A);
```

This fetches the latest version and metadata of the resource referenced by the resource instance's `Id` property.

[operationoutcome]: http://www.hl7.org/fhir/operationoutcome.html
