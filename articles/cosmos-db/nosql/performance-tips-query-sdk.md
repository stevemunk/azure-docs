---
title: Azure Cosmos DB performance tips for queries using the Azure Cosmos DB SDK
description: Learn query configuration options to help improve performance using the Azure Cosmos DB SDK.
author: ealsur
ms.service: cosmos-db
ms.subservice: nosql
ms.topic: how-to
ms.date: 06/20/2023
ms.author: maquaran
ms.devlang: csharp, java
ms.custom: devx-track-java, devx-track-extended-java
zone_pivot_groups: programming-languages-set-cosmos
---

# Query performance tips for Azure Cosmos DB SDKs
[!INCLUDE[NoSQL](../includes/appliesto-nosql.md)]

Azure Cosmos DB is a fast, flexible distributed database that scales seamlessly with guaranteed latency and throughput levels. You don't have to make major architecture changes or write complex code to scale your database with Azure Cosmos DB. Scaling up and down is as easy as making a single API call. To learn more, see [provision container throughput](how-to-provision-container-throughput.md) or [provision database throughput](how-to-provision-database-throughput.md).

::: zone pivot="programming-language-csharp"

## Reduce Query Plan calls

To execute a query, a query plan needs to be built. This in general represents a network request to the Azure Cosmos DB Gateway, which adds to the latency of the query operation. There are two ways to remove this request and reduce the latency of the query operation:

### Use local Query Plan generation

The SQL SDK includes a native ServiceInterop.dll to parse and optimize queries locally. ServiceInterop.dll is supported only on the **Windows x64** platform. The following types of applications use 32-bit host processing by default. To change host processing to 64-bit processing, follow these steps, based on the type of your application:

- For executable applications, you can change host processing by setting the [platform target](/visualstudio/ide/how-to-configure-projects-to-target-platforms?preserve-view=true) to **x64**  in the **Project Properties** window, on the **Build** tab.

- For VSTest-based test projects, you can change host processing by selecting **Test** > **Test Settings** > **Default Processor Architecture as X64** on the Visual Studio **Test** menu.

- For locally deployed ASP.NET web applications, you can change host processing by selecting **Use the 64-bit version of IIS Express for web sites and projects** under **Tools** > **Options** > **Projects and Solutions** > **Web Projects**.

- For ASP.NET web applications deployed on Azure, you can change host processing by selecting the **64-bit** platform in **Application settings** in the Azure portal.

> [!NOTE] 
> By default, new Visual Studio projects are set to **Any CPU**. We recommend that you set your project to **x64** so it doesn't switch to **x86**. A project set to **Any CPU** can easily switch to **x86** if an x86-only dependency is added.<br/>
> ServiceInterop.dll needs to be in the folder that the SDK DLL is being executed from. This should be a concern only if you manually copy DLLs or have custom build/deployment systems.

### Use single partition queries

# [V3 .NET SDK](#tab/v3)

For queries that target a Partition Key by setting the [PartitionKey](/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.partitionkey) property in `QueryRequestOptions` and contain no aggregations (including Distinct, DCount, Group By):

```cs
using (FeedIterator<MyItem> feedIterator = container.GetItemQueryIterator<MyItem>(
    "SELECT * FROM c WHERE c.city = 'Seattle'",
    requestOptions: new QueryRequestOptions() { PartitionKey = new PartitionKey("Washington")}))
{
    // ...
}
```

# [V2 .NET SDK](#tab/v2)

For queries that target a Partition Key by setting the [PartitionKey](/dotnet/api/microsoft.azure.documents.client.feedoptions.partitionkey) property in `FeedOptions` and contain no aggregations (including Distinct, DCount, Group By):

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PartitionKey = new PartitionKey("Washington")
    }).AsDocumentQuery();
```

---

> [!NOTE]
> Cross-partition queries require the SDK to visit all existing partitions to check for results. The more [physical partitions](../partitioning-overview.md#physical-partitions) the container has, the slower they can potentially be.

### Avoid recreating the iterator unnecessarily

When all the query results are consumed by the current component, you don't need to re-create the iterator with the continuation for every page. Always prefer to drain the query fully unless the pagination is controlled by another calling component:

# [V3 .NET SDK](#tab/v3)

```cs
using (FeedIterator<MyItem> feedIterator = container.GetItemQueryIterator<MyItem>(
    "SELECT * FROM c WHERE c.city = 'Seattle'",
    requestOptions: new QueryRequestOptions() { PartitionKey = new PartitionKey("Washington")}))
{
    while (feedIterator.HasMoreResults) 
    {
        foreach(MyItem document in await feedIterator.ReadNextAsync())
        {
            // Iterate through documents
        }
    }
}
```

# [V2 .NET SDK](#tab/v2)

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        PartitionKey = new PartitionKey("Washington")
    }).AsDocumentQuery();
while (query.HasMoreResults) 
{
    foreach(Document document in await queryable.ExecuteNextAsync())
    {
        // Iterate through documents
    }
}
```

---

## Tune the degree of parallelism

# [V3 .NET SDK](#tab/v3)

For queries, tune the [MaxConcurrency](/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxconcurrency) property in `QueryRequestOptions` to identify the best configurations for your application, especially if you perform cross-partition queries (without a filter on the partition-key value). `MaxConcurrency` controls the maximum number of parallel tasks, that is, the maximum of partitions to be visited in parallel. Setting the value to -1 will let the SDK decide the optimal concurrency.

```cs
using (FeedIterator<MyItem> feedIterator = container.GetItemQueryIterator<MyItem>(
    "SELECT * FROM c WHERE c.city = 'Seattle'",
    requestOptions: new QueryRequestOptions() { 
        PartitionKey = new PartitionKey("Washington"),
        MaxConcurrency = -1 }))
{
    // ...
}
```

# [V2 .NET SDK](#tab/v2)

For queries, tune the [MaxDegreeOfParallelism](/dotnet/api/microsoft.azure.documents.client.feedoptions.maxdegreeofparallelism) property in `FeedOptions` to identify the best configurations for your application, especially if you perform cross-partition queries (without a filter on the partition-key value). `MaxDegreeOfParallelism` controls the maximum number of parallel tasks, that is, the maximum of partitions to be visited in parallel. Setting the value to -1 will let the SDK decide the optimal concurrency.

```cs
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT * FROM c WHERE c.city = 'Seattle'", 
    new FeedOptions 
    { 
        MaxDegreeOfParallelism = -1, 
        EnableCrossPartitionQuery = true 
    }).AsDocumentQuery();
```

---

Let's assume that
* D = Default Maximum number of parallel tasks (= total number of processors in the client machine)
* P = User-specified maximum number of parallel tasks
* N = Number of partitions that needs to be visited for answering a query

Following are implications of how the parallel queries would behave for different values of P.
* (P == 0) => Serial Mode
* (P == 1) => Maximum of one task
* (P > 1) => Min (P, N) parallel tasks
* (P < 1) => Min (N, D) parallel tasks

## Tune the page size

When you issue a SQL query, the results are returned in a segmented fashion if the result set is too large.

> [!NOTE] 
> The `MaxItemCount` property shouldn't be used just for pagination. Its main use is to improve the performance of queries by reducing the maximum number of items returned in a single page. 

# [V3 .NET SDK](#tab/v3)

You can also set the page size by using the available Azure Cosmos DB SDKs. The [MaxItemCount](/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount) property in `QueryRequestOptions` allows you to set the maximum number of items to be returned in the enumeration operation. When `MaxItemCount` is set to -1, the SDK automatically finds the optimal value, depending on the document size. For example:

```cs
using (FeedIterator<MyItem> feedIterator = container.GetItemQueryIterator<MyItem>(
    "SELECT * FROM c WHERE c.city = 'Seattle'",
    requestOptions: new QueryRequestOptions() { 
        PartitionKey = new PartitionKey("Washington"),
        MaxItemCount = 1000}))
{
    // ...
}
```

# [V2 .NET SDK](#tab/v2)

You can also set the page size by using the available Azure Cosmos DB SDKs. The [MaxItemCount](/dotnet/api/microsoft.azure.documents.client.feedoptions.maxitemcount) property in `FeedOptions` allows you to set the maximum number of items to be returned in the enumeration operation. When `MaxItemCount` is set to -1, the SDK automatically finds the optimal value, depending on the document size. For example:

```csharp
IQueryable<dynamic> authorResults = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT p.Author FROM Pages p WHERE p.Title = 'About Seattle'", 
    new FeedOptions { MaxItemCount = 1000 });
```

---

When a query is executed, the resulting data is sent within a TCP packet. If you specify too low a value for `MaxItemCount`, the number of trips required to send the data within the TCP packet is high, which affects performance. So if you're not sure what value to set for the `MaxItemCount` property, it's best to set it to -1 and let the SDK choose the default value.

## Tune the buffer size

# [V3 .NET SDK](#tab/v3)

Parallel query is designed to pre-fetch results while the current batch of results is being processed by the client. This pre-fetching helps improve the overall latency of a query. The [MaxBufferedItemCount](/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxbuffereditemcount) property in `QueryRequestOptions` limits the number of pre-fetched results. Set `MaxBufferedItemCount` to the expected number of results returned (or a higher number) to allow the query to receive the maximum benefit from pre-fetching. If you set this value to -1, the system will automatically determine the number of items to buffer.

```cs
using (FeedIterator<MyItem> feedIterator = container.GetItemQueryIterator<MyItem>(
    "SELECT * FROM c WHERE c.city = 'Seattle'",
    requestOptions: new QueryRequestOptions() { 
        PartitionKey = new PartitionKey("Washington"),
        MaxBufferedItemCount = -1}))
{
    // ...
}
```

# [V2 .NET SDK](#tab/v2)

Parallel query is designed to pre-fetch results while the current batch of results is being processed by the client. This pre-fetching helps improve the overall latency of a query. The [MaxBufferedItemCount](/dotnet/api/microsoft.azure.documents.client.feedoptions.maxbuffereditemcount) property in `FeedOptions` limits the number of pre-fetched results. Set `MaxBufferedItemCount` to the expected number of results returned (or a higher number) to allow the query to receive the maximum benefit from pre-fetching. If you set this value to -1, the system will automatically determine the number of items to buffer.

```csharp
IQueryable<dynamic> authorResults = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName), 
    "SELECT p.Author FROM Pages p WHERE p.Title = 'About Seattle'", 
    new FeedOptions { MaxBufferedItemCount = -1 });
```

---

Pre-fetching works the same way regardless of the degree of parallelism, and there's a single buffer for the data from all partitions.



## Next steps

To learn more about performance using the .NET SDK:

* [Best practices for Azure Cosmos DB .NET SDK](best-practice-dotnet.md)
* [Performance tips for Azure Cosmos DB .NET V3 SDK](performance-tips-dotnet-sdk-v3.md)
* [Performance tips for Azure Cosmos DB .NET V2 SDK](performance-tips.md)

::: zone-end
::: zone pivot="programming-language-java"

## Reduce Query Plan calls

To execute a query, a query plan needs to be built. This in general represents a network request to the Azure Cosmos DB Gateway, which adds to the latency of the query operation.

### Use Query Plan caching

The query plan, for a query scoped to a single partition, is cached on the client. This eliminates the need to make a call to the gateway to retrieve the query plan after the first call. The key for the cached query plan is the SQL query string. You need to **make sure the query is [parametrized](query/parameterized-queries.md)**. If not, the query plan cache lookup will often be a cache miss as the query string is unlikely to be identical across calls. Query plan caching is **enabled by default for Java SDK version 4.20.0 and above** and **for Spring Data Azure Cosmos DB SDK version 3.13.0 and above**.

### Use parametrized single partition queries

For parametrized queries that are scoped to a partition key with [setPartitionKey](/java/api/com.azure.cosmos.models.cosmosqueryrequestoptions.setpartitionkey) in `CosmosQueryRequestOptions` and contain no aggregations (including Distinct, DCount, Group By), the query plan can be avoided:

```java
CosmosQueryRequestOptions options = new CosmosQueryRequestOptions();
options.setPartitionKey(new PartitionKey("Washington"));

ArrayList<SqlParameter> paramList = new ArrayList<SqlParameter>();
paramList.add(new SqlParameter("@city", "Seattle"));
SqlQuerySpec querySpec = new SqlQuerySpec(
        "SELECT * FROM c WHERE c.city = @city",
        paramList);

//  Sync API
CosmosPagedIterable<MyItem> filteredItems = 
    container.queryItems(querySpec, options, MyItem.class);

//  Async API
CosmosPagedFlux<MyItem> filteredItems = 
    asyncContainer.queryItems(querySpec, options, MyItem.class);
```

> [!NOTE]
> Cross-partition queries require the SDK to visit all existing partitions to check for results. The more [physical partitions](../partitioning-overview.md#physical-partitions) the container has, the slowed they can potentially be.

## Tune the degree of parallelism

Parallel queries work by querying multiple partitions in parallel. However, data from an individual partitioned container is fetched serially with respect to the query. So, use [setMaxDegreeOfParallelism](/java/api/com.azure.cosmos.models.cosmosqueryrequestoptions.setmaxdegreeofparallelism) on `CosmosQueryRequestOptions` to set the value to the number of partitions you have. If you don't know the number of partitions, you can use `setMaxDegreeOfParallelism` to set a high number, and the system chooses the minimum (number of partitions, user provided input) as the maximum degree of parallelism. Setting the value to -1 will let the SDK decide the optimal concurrency.

It is important to note that parallel queries produce the best benefits if the data is evenly distributed across all partitions with respect to the query. If the partitioned container is partitioned such a way that all or a majority of the data returned by a query is concentrated in a few partitions (one partition in worst case), then the performance of the query would be degraded.

```java
CosmosQueryRequestOptions options = new CosmosQueryRequestOptions();
options.setPartitionKey(new PartitionKey("Washington"));
options.setMaxDegreeOfParallelism(-1);

// Define the query

//  Sync API
CosmosPagedIterable<MyItem> filteredItems = 
    container.queryItems(querySpec, options, MyItem.class);

//  Async API
CosmosPagedFlux<MyItem> filteredItems = 
    asyncContainer.queryItems(querySpec, options, MyItem.class);
```

Let's assume that
* D = Default Maximum number of parallel tasks (= total number of processors in the client machine)
* P = User-specified maximum number of parallel tasks
* N = Number of partitions that needs to be visited for answering a query

Following are implications of how the parallel queries would behave for different values of P.
* (P == 0) => Serial Mode
* (P == 1) => Maximum of one task
* (P > 1) => Min (P, N) parallel tasks
* (P == -1) => Min (N, D) parallel tasks

## Tune the page size

When you issue a SQL query, the results are returned in a segmented fashion if the result set is too large. By default, results are returned in chunks of 100 items or 4 MB, whichever limit is hit first. Increasing the page size will reduce the number of round trips required and increase performance for queries that return more than 100 items. If you're not sure what value to set, 1000 is typically a good choice. Memory consumption will increase as page size increases, so if your workload is memory sensitive consider a lower value.

You can use the `pageSize` parameter in `iterableByPage()` for sync API and `byPage()` for async API, to define a page size:

```java
//  Sync API
Iterable<FeedResponse<MyItem>> filteredItemsAsPages =
    container.queryItems(querySpec, options, MyItem.class).iterableByPage(continuationToken,pageSize);

for (FeedResponse<MyItem> page : filteredItemsAsPages) {
    for (MyItem item : page.getResults()) {
        //...
    }
}

//  Async API
Flux<FeedResponse<MyItem>> filteredItemsAsPages =
    asyncContainer.queryItems(querySpec, options, MyItem.class).byPage(continuationToken,pageSize);

filteredItemsAsPages.map(page -> {
    for (MyItem item : page.getResults()) {
        //...
    }
}).subscribe();
```

## Tune the buffer size

Parallel query is designed to pre-fetch results while the current batch of results is being processed by the client. The pre-fetching helps in overall latency improvement of a query. [setMaxBufferedItemCount](/java/api/com.azure.cosmos.models.cosmosqueryrequestoptions.setmaxbuffereditemcount) in `CosmosQueryRequestOptions` limits the number of pre-fetched results. To maximize the pre-fetching, set the `maxBufferedItemCount` to a higher number than the `pageSize` (NOTE: This can also result in high memory consumption). To minimize the pre-fetching, set the `maxBufferedItemCount` equal to the `pageSize`. If you set this value to 0, the system will automatically determine the number of items to buffer.

```java
CosmosQueryRequestOptions options = new CosmosQueryRequestOptions();
options.setPartitionKey(new PartitionKey("Washington"));
options.setMaxBufferedItemCount(-1);

// Define the query

//  Sync API
CosmosPagedIterable<MyItem> filteredItems = 
    container.queryItems(querySpec, options, MyItem.class);

//  Async API
CosmosPagedFlux<MyItem> filteredItems = 
    asyncContainer.queryItems(querySpec, options, MyItem.class);
```

Pre-fetching works the same way regardless of the degree of parallelism, and there's a single buffer for the data from all partitions.

## Next steps

To learn more about performance using the Java SDK:

* [Best practices for Azure Cosmos DB Java V4 SDK](best-practice-java.md)
* [Performance tips for Azure Cosmos DB Java V4 SDK](performance-tips-java-sdk-v4.md)

::: zone-end
