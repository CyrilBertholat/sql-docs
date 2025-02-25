---
title: "sys.sp_help_change_feed (Transact-SQL)"
description: "The sys.sp_help_change_feed system stored procedure monitors the current configuration of Azure Synapse Link or Fabric Mirrored Database."
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: imotiwala, randolphwest, anagha-todalbagi
ms.date: 11/05/2024
ms.service: fabric
ms.subservice: system-objects
ms.topic: "reference"
ms.custom:
  - ignite-2024
f1_keywords:
  - "sys.sp_help_change_feed_TSQL"
  - "sys.sp_help_change_feed"
  - "sp_help_change_feed_TSQL"
  - "sp_help_change_feed"
helpviewer_keywords:
  - "sp_help_change_feed"
dev_langs:
  - "TSQL"
monikerRange: ">=sql-server-ver16 || =azuresqldb-current || =fabric || =azure-sqldw-latest"
---
# sys.sp_help_change_feed (Transact-SQL)

[!INCLUDE [sqlserver2022-asdb-asa-fabricmirroredsqldb-fabricsqldb](../../includes/applies-to-version/sqlserver2022-asdb-asa-fabricmirroredsqldb-fabricsqldb.md)]

Monitors the current configuration of the change feed.

This system stored procedure is used for:

- The Azure Synapse Link feature for SQL Server instances and Azure SQL Database. For more information, see [Manage Azure Synapse Link for SQL Server and Azure SQL Database](../../sql-server/synapse-link/synapse-link-sql-server-change-feed-manage.md).
- The Fabric Mirrored Database feature for Azure SQL Database. For more information, see [Microsoft Fabric mirrored databases](/fabric/database/mirrored-database/overview).
- SQL database in Microsoft Fabric. For more information, see [SQL database in Microsoft Fabric](/fabric/database/sql/overview).

## Syntax

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

```syntaxsql
EXECUTE sys.sp_help_change_feed;
```

## Result set

| Column name | Data type | Description |
| --- | --- | --- |
| `table_group_id` | **uniqueidentifier** | The unique identifier of the table group. |
| `table_group_name` | **nvarchar(140)** | The name of the table group. |
| `destination_location` | **nvarchar(512)** | URL string of the landing zone folder. |
| `destination_credential` | **sysname** | The credential name to access the landing zone. |
| `workspace_id` | **nvarchar(247)** | The related Synapse workspace Azure resource ID. |
| `synapse_workgroup_name` | **nvarchar(50)** | The related Synapse workspace name. |
| `schema_name` | **sysname** | The database schema name of the change feed table. |
| `table_name` | **sysname** | The name of the change feed table. |
| `table_id` | **uniqueidentifier** | The unique identifier for the change feed table. Generated during change feed setup workflow. |
| `table_object_id` | **int** | The object ID of the change feed table. |
| `state` | **tinyint** | The state of the change feed table. Valid state values: `1` - Enabled. `2` - Exporting. `3` - Exported. `4` - Active. `5` - Disabled. `6` - Pending Disablement. |
| `version` | **binary(10)** | The version of the change feed table. |
| `snapshot_phase` | **tinyint** | Phase of the current snapshot which progresses from one to six. <br/>`1` - ABORT_PRIOR_SNAPSHOT_IF_ANY<br/>`2` - SET_TABLEVERSIONLSN<br/>`3` - EXPORT_SCHEMA_FILE<br/>`4` - EMIT_SNAPSHOT_BEGINENTRY<br/>`5` - EXPORT_DATA_FILE<br/>`6` - EMIT_SNAPSHOT_ENDENTRY|
| `snapshot_current_phase_time` | **datetime** | Time when the current snapshot phase started. |
| `snapshot_retry_count` | **int** | Number of times snapshot has attempted to retry. |
| `snapshot_start_time` | **datetime** | Start time of snapshot phase |
| `snapshot_end_time` | **datetime** | End time of snapshot phase |
| `snapshot_row_count` | **int** | The number of rows of data being exported during the snapshot operation of the change feed table |
| `destination_type` | **int** | `0` = Azure Synapse Link. `2` = Fabric mirroring. |

## Permissions

Currently, only a member of the **sysadmin** server role or **db_owner** role, or a user with **CONTROL** database permissions can execute this procedure.

## Examples

To check the status of tables and metadata:

```sql
EXEC sp_help_change_feed;
```

## Related content

- [sys.sp_help_change_feed_table (Transact-SQL)](sp-help-change-feed-table.md)
- [sys.sp_help_change_feed_table_groups (Transact-SQL)](sp-help-change-feed-table-groups.md)
- [sys.sp_help_change_feed_settings (Transact-SQL)](sp-help-change-feed-settings.md)
- [sys.sp_change_feed_configure_parameters (Transact-SQL)](sp-change-feed-configure-parameters.md)
- [sys.dm_change_feed_log_scan_sessions (Transact-SQL)](../system-dynamic-management-views/sys-dm-change-feed-log-scan-sessions.md)
- [sys.dm_change_feed_errors (Transact-SQL)](../system-dynamic-management-views/sys-dm-change-feed-errors.md)
