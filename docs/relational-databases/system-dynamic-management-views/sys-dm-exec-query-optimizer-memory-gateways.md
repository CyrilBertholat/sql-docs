---
title: "sys.dm_exec_query_optimizer_memory_gateways (Transact-SQL)"
description: Returns the current status of resource semaphores used to reduce concurrent query optimization.
author: MikeRayMSFT
ms.author: mikeray
ms.reviewer: wiassaf
ms.date: 12/03/2024
ms.service: sql
ms.subservice: performance
ms.topic: "reference"
f1_keywords:
  - "dm_exec_query_optimizer_memory_gateways_TSQL"
  - "dm_exec_query_optimizer_memory_gateways"
  - "sys.dm_exec_query_optimizer_memory_gateways_TSQL"
  - "sys.dm_exec_query_optimizer_memory_gateways"
helpviewer_keywords:
  - "sys.dm_exec_query_optimizer_memory_gateways dynamic management view"
dev_langs:
  - "TSQL"
monikerRange: "=azuresqldb-current || >=sql-server-2016 || >=sql-server-linux-2017 || =azuresqldb-mi-current"
---
# sys.dm_exec_query_optimizer_memory_gateways (Transact-SQL)

[!INCLUDE [sqlserver2016-asdb-asdbmi](../../includes/applies-to-version/sqlserver2016-asdb-asdbmi.md)]

Returns the current status of resource semaphores used to reduce concurrent query optimization.

|Column|Type|Description|  
|----------|---------------|-----------------|  
| `pool_id` |**int**|Resource pool ID under Resource Governor|  
| `name` |**sysname**|Compile gate name (Small Gateway, Medium Gateway, Big Gateway)|
| `max_count` |**int**|The maximum configured count of concurrent compiles|
| `active_count` |**int**|The currently active count of compiles in this gate|
| `waiter_count` |**int**|The number of waiters in this gate|
| `threshold_factor` |**bigint**|Threshold factor which defines the maximum memory portion used by query optimization. For the small gateway, threshold_factor indicates the maximum optimizer memory usage in bytes for one query before it is required to gain access in the small gateway. For the medium and big gateway, threshold_factor shows the portion of total server memory available for this gate. It is used as a divisor when calculating the memory usage threshold for the gate.|
| `threshold` |**bigint**|Next threshold memory in bytes. The query is required to gain access to this gateway if its memory consumption reaches this threshold. `-1` if the query is not required to gain access to this gateway.|
| `is_active` |**bit**|Whether the query is required to pass the current gate or not.|


## Permissions
SQL Server requires VIEW SERVER STATE permission on the server.

Azure SQL Database requires the VIEW DATABASE STATE permission in the database.

### Permissions for SQL Server 2022 and later

Requires VIEW SERVER PERFORMANCE STATE permission on the server.

## Remarks
SQL Server uses a tiered gateway approach to reduce the number of permitted concurrent compilations. Three gateways are used, including small, medium, and big. Gateways help prevent the exhausting of overall memory resources by larger compilation memory-requiring consumers.

Waits on a gateway result in delayed compilation. In addition to delays in compilation, reduced requests will have an associated RESOURCE_SEMAPHORE_QUERY_COMPILE wait type accumulation. The RESOURCE_SEMAPHORE_QUERY_COMPILE wait type might indicate that queries are using a large amount of memory for compilation and that memory is exhausted. Alternatively, there might be sufficient memory available overall but available units in a specific gateway are exhausted. The output of `sys.dm_exec_query_optimizer_memory_gateways` can be used to troubleshoot scenarios where there was insufficient memory to compile a query execution plan.  

## Examples

<a id="a-viewing-statistics-on-resource-semaphores"></a>

### A. View statistics on resource semaphores

What are the current optimizer memory gateway statistics for this instance of SQL Server?

```sql
SELECT [pool_id], [name], [max_count], [active_count],
       [waiter_count], [threshold_factor], [threshold],
       [is_active]
FROM sys.dm_exec_query_optimizer_memory_gateways;   

```  

## Related content

- [System dynamic management views](system-dynamic-management-views.md)
- [Execution Related Dynamic Management Views and Functions (Transact-SQL)](execution-related-dynamic-management-views-and-functions-transact-sql.md)
- [How to use the DBCC MEMORYSTATUS command to monitor memory usage on SQL Server 2005](/troubleshoot/sql/database-engine/performance/dbcc-memorystatus-monitor-memory-usage)
- [Large query compilation waits on RESOURCE_SEMAPHORE_QUERY_COMPILE in SQL Server 2014](https://support.microsoft.com/help/3024815/large-query-compilation-waits-on-resource-semaphore-query-compile-in-sql-server-2014)
