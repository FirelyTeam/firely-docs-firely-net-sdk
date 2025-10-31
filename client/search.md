# Searching for resources

FHIR has extensive support for searching resources through the use of
the REST interface. Describing all the possibilities is outside the
scope of this document, but much more details can be found online in the
[specification](http://www.hl7.org/implement/standards/fhir/search.html).

The FHIR client exposes several operations for performing searches.

## Searching within a specific type of resource

The most basic search is the client's `SearchAsync<T>(string[] criteria = null, string[] includes = null, int? pageSize = null)` method. It searches all resources of a specific type based on zero or more criteria. Criteria must conform to the parameters as they would appear on the search URL in the REST interface. For example, searching for all patients named "Eve":

```csharp
Bundle results = await client.SearchAsync<Patient>(new string[] { "family:exact=Eve" });
```

The search will return a Bundle containing entries for each resource
found. It is even possible to leave out all criteria, effectively
resulting in a search that returns all resources of the given type.
Additionally, there is a `SearchAsync()` overload that does not use the
generic `T` argument, you can pass the type of resource as a string in
the first parameter instead.

## Searching for a resource with a specific id

In some cases you may already have the id of a specific resource (e.g.
an Observation with logical id `123`, corresponding to the url
`Observation/123`). In this case you can use
`SearchById<T>(string id, string[] includes = null, int? pageSize = null)`.

Note that this method still returns a `Bundle`. The operation differs from a `ReadAsync<T>()` call because it can also return included resources. For example, given an id `123` for an `Observation`, you can request the server to return both the `Observation` and its `subject`:

```csharp
var incl = new string[] { "Observation.subject" };
Bundle results = await client.SearchByIdAsync<Observation>("123", incl);
```

## System wide search

Some servers allow you to execute searches across *all* resource types.

This uses the client's `WholeSystemSearchAsync(string[] criteria = null, string[] includes = null, int? pageSize = null)` method. For example:

```csharp
Bundle results = await client.WholeSystemSearchAsync(new string[] { "name=foo" });
```

would then not only return Patients with "foo" in their name, but
Devices named "foo" as well.

## Complex searches

An alternative way to specify a query is by creating a `SearchParam`
resource and pass this to the client's `SearchAsync<TResource>(SearchParam q)` overload. The
`SearchParam` resource has a set of fluent calls to allow you to easily
construct more complex queries:

```csharp
 var q = new SearchParams()
           .Where("name:exact=ewout")
           .OrderBy("birthdate", SortOrder.Descending)
           .SummaryOnly().Include("Patient:organization")
           .LimitTo(20);

Bundle result = await client.SearchAsync<Patient>(q);
```

Note that unlike the search options shown before, you can specify search
ordering and the use of a summary result. As well, this syntax avoids
the need to create arrays of strings as parameters and tends to be more
readable.

## Paged Results

Normally, any FHIR server will limit the number of search results
returned. In the previous example, we explicitly limited the number of
results per page to 20.


The `FhirClient` provides a `ContinueAsync` method to page through a search
result `Bundle` after the first page has been received:

```csharp
var result = await client.SearchAsync(q);

while (result != null)
{
    // Do something useful
    result = await client.ContinueAsync(result);
}
```

Note that `ContinueAsync` supports a second parameter that allows you to
browse forward, backward, or go immediately to the first or last page of
the search result.
