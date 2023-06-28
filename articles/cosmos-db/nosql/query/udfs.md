---
title: User-defined functions (UDFs) in Azure Cosmos DB
description: Learn about User-defined functions in Azure Cosmos DB.
author: seesharprun
ms.service: cosmos-db
ms.subservice: nosql
ms.topic: conceptual
ms.date: 04/09/2020
ms.author: sidandrews
ms.reviewer: jucocchi
ms.custom: ignite-2022
---

# User-defined functions (UDFs) in Azure Cosmos DB
[!INCLUDE[NoSQL](../../includes/appliesto-nosql.md)]

The API for NoSQL provides support for user-defined functions (UDFs). With scalar UDFs, you can pass in zero or many arguments and return a single argument result. The API checks each argument for being legal JSON values.  

## UDF use cases

The API extends the SQL syntax to support custom application logic using UDFs. You can register UDFs with the API for NoSQL, and reference them in SQL queries. Unlike stored procedures and triggers, UDFs are read-only.

Using UDFs, you can extend Azure Cosmos DB's query language. UDFs are a great way to express complex business logic in a query's projection.

However, we recommending avoiding UDFs when:

- An equivalent [system function](system-functions.md) already exists in Azure Cosmos DB. System functions will always use fewer RU's than the equivalent UDF.
- The UDF is the only filter in the `WHERE` clause of your query. UDF's do not utilize the index so evaluating the UDF will require loading documents. Combining additional filter predicates that use the index, in combination with a UDF, in the `WHERE` clause will reduce the number of documents processed by the UDF.

If you must use the same UDF multiple times in a query, you should reference the UDF in a [subquery](subquery.md#evaluate-once-and-reference-many-times), allowing you to use a JOIN expression to evaluate the UDF once but reference it many times.

> [!NOTE]
> Server-side JavaScript features including user-defined functions do not support importing modules.

## Examples

The following example registers a UDF under an item container in the Azure Cosmos DB database. The example creates a UDF whose name is `REGEX_MATCH`. It accepts two JSON string values, `input` and `pattern`, and checks if the first matches the pattern specified in the second using JavaScript's `string.match()` function.

```javascript
       UserDefinedFunction regexMatchUdf = new UserDefinedFunction
       {
           Id = "REGEX_MATCH",
           Body = @"function (input, pattern) {
                      return input.match(pattern) !== null;
                   };",
       };

       UserDefinedFunction createdUdf = client.CreateUserDefinedFunctionAsync(
           UriFactory.CreateDocumentCollectionUri("myDatabase", "families"),
           regexMatchUdf).Result;  
```

Now, use this UDF in a query projection. You must qualify UDFs with the case-sensitive prefix `udf.` when calling them from within queries.

```sql
    SELECT udf.REGEX_MATCH(Families.address.city, ".*eattle")
    FROM Families
```

The results are:

```json
    [
      {
        "$1": true
      },
      {
        "$1": false
      }
    ]
```

You can use the UDF qualified with the `udf.` prefix inside a filter, as in the following example:

```sql
    SELECT Families.id, Families.address.city
    FROM Families
    WHERE udf.REGEX_MATCH(Families.address.city, ".*eattle")
```

The results are:

```json
    [{
        "id": "AndersenFamily",
        "city": "Seattle"
    }]
```

In essence, UDFs are valid scalar expressions that you can use in both projections and filters.

To expand on the power of UDFs, look at another example with conditional logic:

```javascript
       UserDefinedFunction seaLevelUdf = new UserDefinedFunction()
       {
           Id = "SEALEVEL",
           Body = @"function(city) {
                   switch (city) {
                       case 'Seattle':
                           return 520;
                       case 'NY':
                           return 410;
                       case 'Chicago':
                           return 673;
                       default:
                           return -1;
                    }"
            };

            UserDefinedFunction createdUdf = await client.CreateUserDefinedFunctionAsync(
                UriFactory.CreateDocumentCollectionUri("myDatabase", "families"),
                seaLevelUdf);
```

The following example exercises the UDF:

```sql
    SELECT f.address.city, udf.SEALEVEL(f.address.city) AS seaLevel
    FROM Families f
```

The results are:

```json
     [
      {
        "city": "Seattle",
        "seaLevel": 520
      },
      {
        "city": "NY",
        "seaLevel": 410
      }
    ]
```

If the properties referred to by the UDF parameters aren't available in the JSON value, the parameter is considered as undefined and the UDF invocation is skipped. Similarly, if the result of the UDF is undefined, it's not included in the result.

As the preceding examples show, UDFs integrate the power of JavaScript language with the API for NoSQL. UDFs provide a rich programmable interface to do complex procedural, conditional logic with the help of built-in JavaScript runtime capabilities. The API for NoSQL provides the arguments to the UDFs for each source item at the current WHERE or SELECT clause stage of processing. The result is seamlessly incorporated in the overall execution pipeline. In summary, UDFs are great tools to do complex business logic as part of queries.

## Next steps

- [Introduction to Azure Cosmos DB](../../introduction.md)
- [System functions](system-functions.md)
- [Aggregates](aggregate-functions.md)
