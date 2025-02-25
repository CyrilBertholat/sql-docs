---
title: "Context Connection"
description: In Microsoft SQL Server, the context connection lets you run Transact-SQL statements in the same context where your code was invoked.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
helpviewer_keywords:
  - "context connections [CLR integration]"
  - "database objects [CLR integration], connections"
  - "connections [CLR integration]"
  - "context [CLR integration]"
dev_langs:
  - "VB"
  - "CSharp"
---
# Context connection

[!INCLUDE [SQL Server](../../../includes/applies-to-version/sqlserver.md)]

The problem of internal data access is a fairly common scenario. That is, you wish to access the same server on which your common language runtime (CLR) stored procedure or function is executing. One option is to create a connection using `System.Data.SqlClient.SqlConnection`, specify a connection string that points to the local server, and open the connection. This method requires specifying credentials for logging in. The connection is in a different database session than the stored procedure or function, it might have different `SET` options, it's in a separate transaction, it doesn't see your temporary tables, and so on.

If your managed stored procedure or function code is executing in the SQL Server process, it's because someone connected to that server and executed a SQL statement to invoke it. You probably want the stored procedure or function to execute in the context of that connection, along with its transaction, `SET` options, and so on. This is called the context connection.

The context connection lets you execute Transact-SQL statements in the same context that your code was invoked in the first place. In order to obtain the context connection, you must use the "context connection" connection string keyword, as in the following example.

### [C#](#tab/csharp)

```csharp
using(SqlConnection connection = new SqlConnection("context connection=true"))
{
    connection.Open();
    // Use the connection
}
```

### [Visual Basic .NET](#tab/vb)

```vb
Using connection as new SqlConnection("context connection=true")
    connection.Open()
    ' Use the connection
End Using
```

---

## In this section

| Article | Description |
| --- | --- |
| [Context connections vs. regular connections](context-connections-vs-regular-connections.md) | Describes the differences between regular and context connections. |
| [Restrictions on context connections and regular connections](context-connections-and-regular-connections-restrictions.md) | Describes the restrictions on regular and context connections. |
