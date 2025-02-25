---
title: "Invoking CLR User-Defined Aggregate Functions"
description: In SQL Server CLR integration, use Transact-SQL SELECT to invoke CLR user-defined aggregates, subject to the rules that apply to system aggregate functions.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
helpviewer_keywords:
  - "aggregate functions [CLR integration]"
  - "invoking user-defined aggregate functions"
  - "user-defined functions [CLR integration]"
dev_langs:
  - "TSQL"
  - "VB"
  - "CSharp"
---
# Invoke CLR user-defined aggregate functions

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

In [!INCLUDE [tsql](../../includes/tsql-md.md)] `SELECT` statements, you can invoke common language runtime (CLR) user-defined aggregates, subject to all the rules that apply to system aggregate functions.

The following additional rules apply:

- The current user must have `EXECUTE` permission on the user-defined aggregate.

- User-defined aggregates must be invoked using a two-part name in the form of <schema_name>.<udagg_name>.

- The argument type of the user-defined aggregate must match or be implicitly convertible to the *input_type* of the aggregate, as defined in the `CREATE AGGREGATE` statement.

- The return type of the user-defined aggregate must match the *return_type* in the `CREATE AGGREGATE` statement.

## Examples

### A. User-defined aggregate concatenating string values

The following code is an example of a user-defined aggregate function that concatenates a set of string values taken from a column in a table:

### [C#](#tab/csharp)

```csharp
using System;
using System.Data;
using Microsoft.SqlServer.Server;
using System.Data.SqlTypes;
using System.IO;
using System.Text;

[Serializable]
[SqlUserDefinedAggregate(
    Format.UserDefined, //use clr serialization to serialize the intermediate result
    IsInvariantToNulls = true, //optimizer property
    IsInvariantToDuplicates = false, //optimizer property
    IsInvariantToOrder = false, //optimizer property
    MaxByteSize = 8000) //maximum size in bytes of persisted value
]
public class Concatenate : IBinarySerialize
{
    /// <summary>
    /// The variable that holds the intermediate result of the concatenation
    /// </summary>
    public StringBuilder intermediateResult;

    /// <summary>
    /// Initialize the internal data structures
    /// </summary>
    public void Init()
    {
        this.intermediateResult = new StringBuilder();
    }

    /// <summary>
    /// Accumulate the next value, not if the value is null
    /// </summary>
    /// <param name="value"></param>
    public void Accumulate(SqlString value)
    {
        if (value.IsNull)
        {
            return;
        }

        this.intermediateResult.Append(value.Value).Append(',');
    }

    /// <summary>
    /// Merge the partially computed aggregate with this aggregate.
    /// </summary>
    /// <param name="other"></param>
    public void Merge(Concatenate other)
    {
        this.intermediateResult.Append(other.intermediateResult);
    }

    /// <summary>
    /// Called at the end of aggregation, to return the results of the aggregation.
    /// </summary>
    /// <returns></returns>
    public SqlString Terminate()
    {
        string output = string.Empty;
        //delete the trailing comma, if any
        if (this.intermediateResult != null
            && this.intermediateResult.Length > 0)
        {
            output = this.intermediateResult.ToString(0, this.intermediateResult.Length - 1);
        }

        return new SqlString(output);
    }

    public void Read(BinaryReader r)
    {
        intermediateResult = new StringBuilder(r.ReadString());
    }

    public void Write(BinaryWriter w)
    {
        w.Write(this.intermediateResult.ToString());
    }
}
```

### [Visual Basic .NET](#tab/vb)

```vb
Imports System
Imports System.Data
Imports Microsoft.SqlServer.Server
Imports System.Data.SqlTypes
Imports System.IO
Imports System.Text

<Serializable(), SqlUserDefinedAggregate(Format.UserDefined, IsInvariantToNulls:=True, IsInvariantToDuplicates:=False, IsInvariantToOrder:=False, MaxByteSize:=8000)> _
Public Class Concatenate
    Implements IBinarySerialize

    ''' <summary>
    ''' The variable that holds the intermediate result of the concatenation
    ''' </summary>
    Public intermediateResult As StringBuilder

    ''' <summary>
    ''' Initialize the internal data structures
    ''' </summary>
    Public Sub Init()
        Me.intermediateResult = New StringBuilder()
    End Sub

    ''' <summary>
    ''' Accumulate the next value, not if the value is null
    ''' </summary>
    ''' <param name="value"></param>
    Public Sub Accumulate(ByVal value As SqlString)
        If value.IsNull Then
            Return
        End If

        Me.intermediateResult.Append(value.Value).Append(","c)
    End Sub
    ''' <summary>
    ''' Merge the partially computed aggregate with this aggregate.
    ''' </summary>
    ''' <param name="other"></param>
    Public Sub Merge(ByVal other As Concatenate)
        Me.intermediateResult.Append(other.intermediateResult)
    End Sub

    ''' <summary>
    ''' Called at the end of aggregation, to return the results of the aggregation.
    ''' </summary>
    ''' <returns></returns>
    Public Function Terminate() As SqlString
        Dim output As String = String.Empty

        'delete the trailing comma, if any
        If Not (Me.intermediateResult Is Nothing) AndAlso Me.intermediateResult.Length > 0 Then
            output = Me.intermediateResult.ToString(0, Me.intermediateResult.Length - 1)
        End If

        Return New SqlString(output)
    End Function

    Public Sub Read(ByVal r As BinaryReader) Implements IBinarySerialize.Read
        intermediateResult = New StringBuilder(r.ReadString())
    End Sub

    Public Sub Write(ByVal w As BinaryWriter) Implements IBinarySerialize.Write
        w.Write(Me.intermediateResult.ToString())
    End Sub
End Class
```

---

Once you compile the code into `MyAgg.dll`, you can register the aggregate in [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] as follows:

```sql
CREATE ASSEMBLY MyAgg
    FROM 'C:\MyAgg.dll';
GO

CREATE AGGREGATE MyAgg(@input NVARCHAR (200))
    RETURNS NVARCHAR (MAX)
    EXTERNAL NAME MyAgg.Concatenate;
```

> [!NOTE]  
> Visual C++ database objects, such as scalar-valued functions, that have been compiled with the `/clr:pure` compiler option aren't supported for execution in [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)].

As with most aggregates, the bulk of the logic is in the `Accumulate` method. Here, the string that is passed in as a parameter to the `Accumulate` method is appended to the `StringBuilder` object that was initialized in the `Init` method. Assuming that the `Accumulate` method wasn't already called, a comma is also appended to the `StringBuilder` before appending the passed-in string. At the conclusion of the computational tasks, the `Terminate` method is called, which returns the `StringBuilder` as a string.

For example, consider a table with the following schema:

```sql
CREATE TABLE BookAuthors
(
    BookID INT NOT NULL,
    AuthorName NVARCHAR (200) NOT NULL
);
```

Then insert the following rows:

```sql
INSERT BookAuthors
VALUES
    (1, 'Johnson'),
    (2, 'Taylor'),
    (3, 'Steven'),
    (2, 'Mayler'),
    (3, 'Roberts'),
    (3, 'Michaels');
```

The following query would then produce the following result:

```sql
SELECT BookID, dbo.MyAgg(AuthorName)
FROM BookAuthors
GROUP BY BookID;
```

| BookID | Author Names |
| --- | --- |
| `1` | `Johnson` |
| `2` | `Taylor, Mayler` |
| `3` | `Roberts, Michaels, Steven` |

### B. User-defined aggregate with two parameters

The following sample shows an aggregate that has two parameters on the `Accumulate` method.

### [C#](#tab/csharp)

```csharp
using System;
using System.Data;
using System.Data.SqlClient;
using System.Data.SqlTypes;
using Microsoft.SqlServer.Server;

[Serializable]
[SqlUserDefinedAggregate(
    Format.Native,
    IsInvariantToDuplicates = false,
    IsInvariantToNulls = true,
    IsInvariantToOrder = true,
    IsNullIfEmpty = true,
    Name = "WeightedAvg")]
public struct WeightedAvg
{
    /// <summary>
    /// The variable that holds the intermediate sum of all values multiplied by their weight
    /// </summary>
    private long sum;

    /// <summary>
    /// The variable that holds the intermediate sum of all weights
    /// </summary>
    private int count;

    /// <summary>
    /// Initialize the internal data structures
    /// </summary>
    public void Init()
    {
        sum = 0;
        count = 0;
    }

    /// <summary>
    /// Accumulate the next value, not if the value is null
    /// </summary>
    /// <param name="Value">Next value to be aggregated</param>
    /// <param name="Weight">The weight of the value passed to Value parameter</param>
    public void Accumulate(SqlInt32 Value, SqlInt32 Weight)
    {
        if (!Value.IsNull && !Weight.IsNull)
        {
            sum += (long)Value * (long)Weight;
            count += (int)Weight;
        }
    }

    /// <summary>
    /// Merge the partially computed aggregate with this aggregate
    /// </summary>
    /// <param name="Group">The other partial results to be merged</param>
    public void Merge(WeightedAvg Group)
    {
        sum += Group.sum;
        count += Group.count;
    }

    /// <summary>
    /// Called at the end of aggregation, to return the results of the aggregation.
    /// </summary>
    /// <returns>The weighted average of all inputed values</returns>
    public SqlInt32 Terminate()
    {
        if (count > 0)
        {
            int value = (int)(sum / count);
            return new SqlInt32(value);
        }
        else
        {
            return SqlInt32.Null;
        }
    }
}
```

### [Visual Basic .NET](#tab/vb)

```vb
Imports System
Imports System.Data
Imports System.Data.SqlClient
Imports System.Data.SqlTypes
Imports Microsoft.SqlServer.Server
Imports System.Runtime.InteropServices

<StructLayout(LayoutKind.Sequential)> _
<Serializable(), SqlUserDefinedAggregate(Format.Native, _
IsInvariantToDuplicates:=False, _
IsInvariantToNulls:=True, _
IsInvariantToOrder:=True, _
IsNullIfEmpty:=True, _
Name:="WeightedAvg")> _
Public Class WeightedAvg

    ''' <summary>
    ''' The variable that holds the intermediate sum of all values multiplied by their weight
    ''' </summary>
    Private sum As Long

    ''' <summary>
    ''' The variable that holds the intermediate sum of all weights
    ''' </summary>
    Private count As Integer

    ''' <summary>
    ''' The variable that holds the intermediate sum of all weights
    ''' </summary>
    Public Sub Init()
        sum = 0
        count = 0
    End Sub

    ''' <summary>
    ''' Accumulate the next value, not if the value is null
    ''' </summary>
    ''' <param name="Value">Next value to be aggregated</param>
    ''' <param name="Weight">The weight of the value passed to Value parameter</param>
    Public Sub Accumulate(ByVal Value As SqlInt32, ByVal Weight As SqlInt32)
        If Not Value.IsNull AndAlso Not Weight.IsNull Then
            sum += CType(Value, Long) * CType(Weight, Long)
            count += CType(Weight, Integer)
        End If
    End Sub

    ''' <summary>
    ''' Merge the partially computed aggregate with this aggregate.
    ''' </summary>
    ''' <param name="Group">The other partial results to be merged</param>
    Public Sub Merge(ByVal Group As WeightedAvg)
        sum = Group.sum
        count = Group.count
    End Sub

    ''' <summary>
    ''' Called at the end of aggregation, to return the results of the aggregation.
    ''' </summary>
    ''' <returns>The weighted average of all inputed values</returns>
    Public Function Terminate() As SqlInt32
        If count > 0 Then
            ''                        int value = (int)(sum / count);
            ''          return new SqlInt32(value);
            Dim value As Integer = CType(sum / count, Integer)
            Return New SqlInt32(value)
        Else
            Return SqlInt32.Null
        End If
    End Function
End Class
```

---

After you compile the [!INCLUDE [c-sharp-md](../../includes/c-sharp-md.md)] or [!INCLUDE [visual-basic-md](../../includes/visual-basic-md.md)] .NET source code, run the following [!INCLUDE [tsql](../../includes/tsql-md.md)]. This script assumes that the DLL is called WghtAvg.dll and is in the root directory of your C drive. A database called test is also assumed.

```sql
USE test;
GO

-- EXECUTE sp_configure 'clr enabled', 1;
-- RECONFIGURE WITH OVERRIDE;
-- GO
IF EXISTS (SELECT name
           FROM systypes
           WHERE name = 'MyTableType')
    DROP TYPE MyTableType;
GO

IF EXISTS (SELECT name
           FROM sysobjects
           WHERE name = 'WeightedAvg')
    DROP AGGREGATE WeightedAvg;
GO

IF EXISTS (SELECT name
           FROM sys.assemblies
           WHERE name = 'MyClrCode')
    DROP ASSEMBLY MyClrCode;
GO

CREATE ASSEMBLY MyClrCode
    FROM 'C:\WghtAvg.dll';
GO

CREATE AGGREGATE WeightedAvg(@value INT, @weight INT)
    RETURNS INT
    EXTERNAL NAME MyClrCode.WeightedAvg;
GO

CREATE TYPE MyTableType AS TABLE (
    ItemValue INT,
    ItemWeight INT);
GO

DECLARE @myTable AS MyTableType;

INSERT INTO @myTable
VALUES (1, 4),
(6, 1);

SELECT dbo.WeightedAvg(ItemValue, ItemWeight)
FROM @myTable;
GO
```

## Related content

- [CLR user-defined aggregates](clr-user-defined-aggregates.md)
