---
title: Handle duplicate data in Azure Data Explorer
description: This topic will show you various approaches to deal with duplicate data when using Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: how-to
ms.date: 12/19/2018

#Customer intent: I want to learn how to deal with duplicate data.
---

# Handle duplicate data in Azure Data Explorer

Devices sending data to the Cloud maintain a local cache of the data. Depending on the data size, the local cache could be storing data for days or even months. You want to safeguard your analytical databases from malfunctioning devices that resend the cached data and cause data duplication in the analytical database. This topic outlines best practices for handling duplicate data for these types of scenarios.

The best solution for data duplication is preventing the duplication. If possible, fix the issue earlier in the data pipeline, which saves costs associated with data movement along the data pipeline and avoids spending resources on coping with duplicate data ingested into the system. However, in situations where the source system can't be modified, there are various ways to deal with this scenario.

## Understand the impact of duplicate data

Monitor the percentage of duplicate data. Once the percentage of duplicate data is discovered, you can analyze the scope of the issue and business impact and choose the appropriate solution.

Sample query to identify the percentage of duplicate records:

```kusto
let _sample = 0.01; // 1% sampling
let _data =
DeviceEventsAll
| where EventDateTime between (datetime('10-01-2018 10:00') .. datetime('10-10-2018 10:00'));
let _totalRecords = toscalar(_data | count);
_data
| where rand()<= _sample
| summarize recordsCount=count() by hash(DeviceId) + hash(EventId) + hash(StationId)  // Use all dimensions that make row unique. Combining hashes can be improved
| summarize duplicateRecords=countif(recordsCount  > 1)
| extend duplicate_percentage = (duplicateRecords / _sample) / _totalRecords  
```

## Solutions for handling duplicate data

### Solution #1: Don't remove duplicate data

Understand your business requirements and tolerance of duplicate data. Some datasets can manage with a certain percentage of duplicate data. If the duplicated data doesn't have major impact, you can ignore its presence. The advantage of not removing the duplicate data is no additional overhead on the ingestion process or query performance.

### Solution #2: Handle duplicate rows during query

Another option is to filter out the duplicate rows in the data during query. The [`arg_max()`](kusto/query/arg-max-aggfunction.md) aggregated function can be used to filter out the duplicate records and return the last record based on the timestamp (or another column). The advantage of using this method is faster ingestion since de-duplication occurs during query time. In addition, all records (including duplicates) are available for auditing and troubleshooting. The disadvantage of using the `arg_max` function is the additional query time and load on the CPU every time the data is queried. Depending on the amount of the data being queried, this solution may become non-functional or memory-consuming and will require switching to other options.

In the following example, we query the last record ingested for a set of columns that determine the unique records:

```kusto
DeviceEventsAll
| where EventDateTime > ago(90d)
| summarize hint.strategy=shuffle arg_max(EventDateTime, *) by DeviceId, EventId, StationId
```

This query can also be placed inside a function instead of directly querying the table:

```kusto
.create function DeviceEventsView
{
DeviceEventsAll
| where EventDateTime > ago(90d)
| summarize arg_max(EventDateTime, *) by DeviceId, EventId, StationId
}
```

### Solution #3: Use materialized views to deduplicate

[Materialized views](kusto/management/materialized-views/materialized-view-overview.md) can be used for deduplication, by using the [any()](./kusto/query/take-any-aggfunction.md)/[arg_min()](kusto/query/arg-min-aggfunction.md)/[arg_max()](kusto/query/arg-max-aggfunction.md) aggregation functions (see example #4 in [materialized view create command](kusto/management/materialized-views/materialized-view-create.md#examples)). 

> [!NOTE]
> Materialized views come with a cost of consuming cluster's resources, which may not be negligible. For more information, see materialized views [performance considerations](kusto/management/materialized-views/materialized-view-overview.md#performance-considerations).

## Summary

Data duplication can be handled in multiple ways. Evaluate the options carefully, taking into account price and performance, to determine the correct method for your business.

## Next steps

> [!div class="nextstepaction"]
> [Write queries for Azure Data Explorer](write-queries.md)
