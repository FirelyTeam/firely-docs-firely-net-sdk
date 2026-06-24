# Paged results

Normally, any FHIR server will limit the number of results returned for
the `history` and `search` interactions. For these interactions you can
also specify the maximum number of results you would want to receive client
side.

The `FhirClient` provides a `ContinueAsync` method to page through a search
or history result `Bundle` after the first page has been received. `ContinueAsync`
supports a second parameter that specifies the direction in which to page:
forward, backward, or directly to the first or last page of the result. The
default direction is to retrieve the next page. The method returns `null`
when there is no link for the requested direction in the provided `Bundle`.

```csharp
while (result != null)
{
    // Do something with the entries in the result Bundle

    // retrieve the next page of results
    result = await client.ContinueAsync(result);
}

// go to the last page with the direction filled in:
var last_page = await client.ContinueAsync(result, PageDirection.Last);
```
