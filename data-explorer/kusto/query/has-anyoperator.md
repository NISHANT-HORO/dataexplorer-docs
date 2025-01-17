---
title: The case-insensitive has_any string operator - Azure Data Explorer
description: This article describes the case-insensitive has_any string operator in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 10/01/2021
---
# has_any operator

Filters a record set for data with any of a set of case-insensitive strings.

For more information about other operators and to determine which operator is most appropriate for your query, see [datatype string operators](datatypes-string-operators.md).

## Performance tips

> [!NOTE]
> Performance depends on the type of search and the structure of the data.

For faster results, use the case-sensitive version of an operator, for example, `has_cs`, not `has`. For best practices, see [Query best practices](best-practices.md).

## Syntax

*T* `|` `where` *col* `has_any` `(`*list of scalar expressions*`)`   
*T* `|` `where` *col* `has_any` `(`*tabular expression*`)`   
 
## Arguments

* *T* - Tabular input whose records are to be filtered.
* *col* - Column to filter.
* *list of expressions* - Comma separated list of scalar or literal expressions
* *tabular expression* - Tabular expression that has a set of values (if expression has multiple columns, the first column is used)

## Returns

Rows in *T* for which the predicate is `true`

## Notes

* The expression list can produce up to `10,000` values.    
* For tabular expressions, the first column of the result set is selected.   

## Examples 

### Use has_any operator with a list 

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
StormEvents 
| where State has_any ("CAROLINA", "DAKOTA", "NEW") 
| summarize count() by State
```

**Output**

|State|count_|
|---|---|
|NEW YORK|1750|
|NORTH CAROLINA|1721|
|SOUTH DAKOTA|1567|
|NEW JERSEY|1044|
|SOUTH CAROLINA|915|
|NORTH DAKOTA|905|
|NEW MEXICO|527|
|NEW HAMPSHIRE|394|

### Use has_any operator with a dynamic array

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let states = dynamic(['south', 'north']);
StormEvents 
| where State has_any (states)
| summarize count() by State
```

**Output**

|State|count_|
|---|---|
|NORTH CAROLINA|1721|
|SOUTH DAKOTA|1567|
|SOUTH CAROLINA|915|
|NORTH DAKOTA|905|
|ATLANTIC SOUTH|193|
|ATLANTIC NORTH|188|