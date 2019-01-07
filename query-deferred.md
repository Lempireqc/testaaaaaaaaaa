# Query Deferred

## Description

There are two types of IQueryable extension methods:

- Deferred Methods: The query expression is modified but the query is not resolved (Select, Where, etc.).
- Immediate Methods: The query expression is modified and the query is resolved (Count, First, etc.).

However, some features like **Query Cache** and **Query Future** cannot be used directly with Immediate Method since the query is already resolved.

**Query Deferred** provides more flexibility to other features.

### All LINQ IQueryable extension methods and overloads are supported:

| Name | Description | Example |
| :--- | :---------- | :------ |
| `DeferredAggregate` | QueryDeferred extension method. Applies an accumulator function over a sequence. | [Try it](#) |
| `DeferredAll` | | [Try it](#) |
| `DeferredAny` | | [Try it](#) |
| `DeferredAverage` | | [Try it](#) |
| `DeferredContains` | | [Try it](#) |
| `DeferredCount` | | [Try it](#) |
| `DeferredElementAt` | | [Try it](#) |
| `DeferredElementAtOrDefault` | | [Try it](#) |
| `DeferredFirst` | | [Try it](#) |
| `DeferredFirstOrDefault` | | [Try it](#) |
| `DeferredLast` | | [Try it](#) |
| `DeferredLastOrDefault` | | [Try it](#) |
| `DeferredLongCount` | | [Try it](#) |
| `DeferredMax` | | [Try it](#) |
| `DeferredMin` | | [Try it](#) |
| `DeferredSequenceEqual` | | [Try it](#) |
| `DeferredSingle` | | [Try it](#) |
| `DeferredSingleOrDefault` | | [Try it](#) |
| `DeferredSum` | | [Try it](#) |

## Real Life Scenarios
### Query Cache
You want to cache the customer count (immediate method) with the [Query Cache](query-cache) feature. You can defer the customer count with the `DeferredCount` method.

```csharp
// ... Coming soon...
```
[Try it](#)

### Query Future
You want to return the customer count (immediate method) with a paged list using the [Query Future](query-future) feature. You can defer the customer count with the `DeferredCount` method.

```csharp
// ... Coming soon...
```
[Try it](#)

## Documnentation

### QueryDeferred<TResult>

###### Methods
| Name | Description | Example |
| :--- | :---------- | :------ |
| `Execute()` | Execute the deferred expression and return the result. | [Try it](#) |
| `ExecuteAsync()` | Execute asynchrounously the deferred expression and return the result. | [Try it](#) |
| `ExecuteAsync(CancellationToken cancellationToken)` | Execute asynchrounously the deferred expression and return the result.  | [Try it](#)  |


