# Retrieving resource history

There are several ways to retrieve version history for resources using the `FhirClient`.

```{note}
Servers are not required to support version retrieval. If the `history` interaction
is supported, the server may choose which levels it supports. You can check the
server's `CapabilityStatement` to see what is available.
```

## History of a specific resource

You can retrieve the version history of a specific resource using the
`HistoryAsync` method of the `FhirClient`. You may specify a date to include only changes made after that date, and a count to limit the number of results returned.

The method returns a `Bundle` resource containing the history for the resource instance. For example:

```csharp
var pat_31_hist = await client.HistoryAsync("Patient/31");
// or
var pat_31_hist = await client.HistoryAsync("Patient/31", new FhirDateTime("2016-11-29").ToDateTimeOffset());
// or
var pat_31_hist = await client.HistoryAsync("Patient/31", new FhirDateTime("2016-11-29").ToDateTimeOffset(), 5);
```

```{note}
The `Bundle` may contain entries without a resource if a version was created by a `delete` interaction.
```


## History for a resource type

Sometimes you may want to retrieve the history for a **type** of resource rather than a single instance (for example, all versions of `Patient` resources). In this case, use the `TypeHistoryAsync` method:

```csharp
var pat_hist = await client.TypeHistoryAsync<Patient>();
```

As with the instance-level method, you can optionally specify a date and page size.


## System-wide history

To retrieve the history for all resources in the system, use the `WholeSystemHistoryAsync` method of the `FhirClient`. You can specify a date and a page size:

```csharp
var lastMonth = DateTime.Today.AddMonths(-1);
var last_month_hist = await client.WholeSystemHistoryAsync(since: lastMonth, pageSize: 10);
```

This function retrieves all changes to all resources made since the specified date and limits the results to the given maximum. See [](paging) for an example of how to page through the resulting `Bundle`.
