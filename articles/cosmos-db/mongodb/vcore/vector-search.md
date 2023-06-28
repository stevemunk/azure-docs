---
title: Vector search on embeddings
titleSuffix: Azure Cosmos DB for MongoDB vCore
description: Use vector indexing and search to integrate AI-based applications in Azure Cosmos DB for MongoDB vCore
author: gahl-levy
ms.author: gahllevy
ms.reviewer: nayakshweta
ms.service: cosmos-db
ms.subservice: mongodb-vcore
ms.custom: build-2023
ms.topic: conceptual
ms.date: 05/10/2023
---

# Using vector search on embeddings in Azure Cosmos DB for MongoDB vCore

[!INCLUDE[MongoDB vCore](../../includes/appliesto-mongodb-vcore.md)]

Use Vector Search in Azure Cosmos DB for MongoDB vCore to seamlessly integrate your AI-based applications, including apps built using [Azure OpenAI embeddings](../../../cognitive-services/openai/tutorials/embeddings.md), with your data stored in Azure Cosmos DB. Vector search enables you to efficiently store, index, and query high dimensional vector data stored directly in Azure Cosmos DB for MongoDB vCore, eliminating the need to transfer your data to more expensive alternatives for vector search capabilities.

## What is Vector search?

Vector search is a method that helps you find similar items based on their data characteristics rather than exact matches on a property field. This technique is useful in applications such as searching for similar text, finding related images, making recommendations, or even detecting anomalies. It works by taking the vector representations (lists of numbers) of your data that you have created using an ML model, or an embeddings API. Examples of embeddings APIs could be [Azure OpenAI Embeddings](/azure/cognitive-services/openai/how-to/embeddings) or [Hugging Face on Azure](https://azure.microsoft.com/solutions/hugging-face-on-azure/). It then measures the distance between the data vectors and your query vector. The data vectors that are closest to your query vector are the ones that are found to be most similar semantically.

By integrating vector search capabilities natively, you can now unlock the full potential of your data in applications built on top of the OpenAI API. You can also create custom-built solutions that use vector embeddings.

## Create a vector index

To create a vector index, use the following createIndex Spec template:

```json
{
  "createIndexes": "<collection_name>",
  "indexes": [
    {
      "name": "<index_name>",
      "key": {
        "<path_to_property>": "cosmosSearch"
      },
      "cosmosSearchOptions": {
        "kind": "vector-ivf",
        "numLists": <integer_value>,
        "similarity": "<string_value>",
        "dimensions": <integer_value>
      }
    }
  ]
}
```

| Field | Type | Description |
| --- | --- | --- |
| `index_name` | `string` | Unique name of the index. |
| `path_to_property` | `string` | Path to the property containing the vector. This path can be a top-level property or a `dot-notation` path to the property. If a `dot-notation` path is used, then all the nonleaf elements can't be arrays. |
| `kind` | `string` | Type of vector index to create. Currently, `vector-ivf` is the only supported index option. |
| `numLists` | `integer` | This integer is the number of clusters the IVF index uses to group the vector data. It's recommended that numLists are set to `rowCount()/1000` for up to a million rows and `sqrt(rowCount)` for more than a million rows. |
| `similarity` | `string` | Similarity metric to use with the IVF index. Possible options are `COS` (cosine distance), `L2` (Euclidean distance) or `IP` (inner product) |
| `dimensions` | `integer` | Number of dimensions for vector similarity. The maximum number of supported dimensions is `2000`. |

In the following examples, we walk through examples on how to index vectors, add documents with vector properties, perform a vector search, and retrieve the index configuration.

### Create a vectorIndex

```javascript
use test;

db.runCommand({
  createIndexes: 'exampleCollection',
  indexes: [
    {
      name: 'vectorSearchIndex',
      key: {
        "vectorContent": "cosmosSearch"
      },
      cosmosSearchOptions: {
        kind: 'vector-ivf',
        numLists: 100,
        similarity: 'COS',
        dimensions: 3
      }
    }
  ]
});
```

This command creates a `vector-ivf` index against the "vectorContent" property in the documents stored in the specified collection, `exampleCollection`. The `cosmosSearchOptions` property specifies the parameters for the IVF vector index. If your document has the vector stored in a nested property, you can set this property using a dot-notation path. For example, `text.vectorContent` if `vectorContent` is a subproperty of `text`.

## Adding vectors to your database

To add vectors to your database's collection, you first need to create the embeddings using your own model, [Azure OpenAI Embeddings](https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/cognitive-services/openai/tutorials/embeddings.md), or another API (such as [Hugging Face on Azure](https://azure.microsoft.com/solutions/hugging-face-on-azure/)). In this example, new documents are added with sample embeddings:

```javascript
db.exampleCollection.insertMany([
  {name: "Eugenia Lopez", bio: "Eugenia is the CEO of AdvenureWorks.", vectorContent: [0.51, 0.12, 0.23]},
  {name: "Cameron Baker", bio: "Cameron Baker CFO of AdvenureWorks.", vectorContent: [0.55, 0.89, 0.44]},
  {name: "Jessie Irwin", bio: "Jessie Irwin is the former CEO of AdventureWorks and now the director of the Our Planet initiative.", vectorContent: [0.13, 0.92, 0.85]},
  {name: "Rory Nguyen", bio: "Rory Nguyen is the founder of AdventureWorks and the president of the Our Planet initiative.", vectorContent: [0.91, 0.76, 0.83]},
]);
```

### Performing a vector search

To perform a vector search, use the `$search` aggregation pipeline stage in a MongoDB query. To use the `cosmosSearch` index, we have introduced a new `cosmosSearch` operator.

```json
{
  "$search": {
    "cosmosSearch": {
        "vector": <vector_to_search>,
        "path": "<path_to_property>",
        "k": <num_results_to_return>
      }
    ...
  }
}
```

### Query a vectorIndex using $search

Continuing with the above example, create another vector, `queryVector`. Vector search measures the distance between `queryVector` and the vectors in the `vectorContent` path of your documents. You can set the number of results the search returns by setting the parameter `k`, which is set to `2` here.

```javascript
const queryVector = [0.52, 0.28, 0.12];
db.exampleCollection.aggregate([
  {
    $search: {
      "cosmosSearch": {
        "vector": queryVector,
        "path": "vectorContent",
        "k": 2
      },
    "returnStoredSource": true
    }
  }
]);
```

In this example, a vector search is performed using `queryVector` as an input via the Mongo shell. The search result is a list of the two most similar items to the query vector, sorted by their similarity scores.

```javascript
[
  {
    _id: ObjectId("645acb54413be5502badff94"),
    name: 'Eugenia Lopez',
    bio: 'Eugenia is the CEO of AdvenureWorks.',
    vectorContent: [ 0.51, 0.12, 0.23 ]
  },
  {
    _id: ObjectId("645acb54413be5502badff97"),
    name: 'Rory Nguyen',
    bio: 'Rory Nguyen is the founder of AdventureWorks and the president of the Our Planet initiative.',
    vectorContent: [ 0.91, 0.76, 0.83 ]
  }
]
```

### Get vector index definitions

To retrieve your vector index definition from the collection, use the `listIndexes` command.

``` javascript
db.exampleCollection.getIndexes();
```

In this example, the vectorIndex is returned along with all the cosmosSearch parameters used to create the index

```javascript
[
  { v: 2, key: { _id: 1 }, name: '_id_', ns: 'test.exampleCollection' },
  {
    v: 2,
    key: { vectorContent: 'cosmosSearch' },
    name: 'vectorSearchIndex',
    cosmosSearch: {
      kind: 'vector-ivf',
      numLists: 100,
      similarity: 'COS',
      dimensions: 3
    },
    ns: 'test.exampleCollection'
  }
]
```

## Features and limitations

* Supported distance metrics: L2 (Euclidean), inner product, and cosine.
* Supported indexing methods: IVFFLAT.
* Indexing vectors up to 2,000 dimensions in size.
* Indexing applies to only one vector per document.

## Next steps

This guide demonstrated how to create a vector index, add documents with vector data, perform a similarity search, and retrieve the index definition. By using vector search, you can efficiently store, index, and query high-dimensional vector data directly in Azure Cosmos DB for MongoDB vCore. Vector search enables you to unlock the full potential of your data with vector embeddings, and empowers you to build more accurate, efficient, and powerful applications.

> [!div class="nextstepaction"]
> [Introduction to Azure Cosmos DB for MongoDB vCore](introduction.md)
