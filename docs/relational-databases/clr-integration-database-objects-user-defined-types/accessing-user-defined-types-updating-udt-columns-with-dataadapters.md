---
title: "Updating UDT Columns With DataAdapters"
description: UDTs in a SQL Server database are supported by using System.Data.DataSet and System.Data.SqlClient.SqlDataAdapter to retrieve and modify data.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
helpviewer_keywords:
  - "ADO.NET [CLR integration]"
  - "updating data [CLR integration]"
  - "populating DataSet"
  - "datasets [CLR integration]"
  - "UDTs [CLR integration], ADO.NET"
  - "updating UDT columns"
  - "user-defined types [CLR integration], ADO.NET"
  - "data adapters [CLR integration]"
dev_langs:
  - "TSQL"
  - "VB"
  - "CSharp"
---
# Update user-defined type (UDT) columns with DataAdapters

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

User-defined types (UDTs) are supported by using a `System.Data.DataSet` and a `System.Data.SqlClient.SqlDataAdapter` to retrieve and modify data.

## Populate a dataset

You can use a [!INCLUDE [tsql](../../includes/tsql-md.md)] `SELECT` statement to select UDT column values to populate a dataset using a data adapter. The following example assumes that you have a `Points` table defined with the following structure and some sample data. The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statements create the `Points` table and insert a few rows.

```sql
CREATE TABLE dbo.Points
(
    id INT PRIMARY KEY,
    p Point
);

INSERT INTO dbo.Points
VALUES (1, CONVERT (Point, '1,3'));

INSERT INTO dbo.Points
VALUES (2, CONVERT (Point, '2,4'));

INSERT INTO dbo.Points
VALUES (3, CONVERT (Point, '3,5'));

INSERT INTO dbo.Points
VALUES (4, CONVERT (Point, '4,6'));
GO
```

The following ADO.NET code fragment retrieves a valid connection string, creates a new `SqlDataAdapter`, and populates a `System.Data.DataTable` with the rows of data from the `Points` table.

### [C#](#tab/csharp)

```sql
SqlDataAdapter da = new SqlDataAdapter(
   "SELECT id, p FROM dbo.Points", connectionString);
DataTable datTable = new DataTable("Points");
da.Fill(datTable);
```

### [Visual Basic .NET](#tab/vb)

```vb
Dim da As New SqlDataAdapter( _
    "SELECT id, p FROM dbo.Points", connectionString)
Dim datTable As New DataTable("Points")
da.Fill(datTable)
```

---

## Update UDT data in a dataset

You can use two methods to update a UDT column in a `DataSet`:

- Provide custom `InsertCommand`, `UpdateCommand`, and `DeleteCommand` objects for a `SqlDataAdapter` object.

- Use the command builder (`System.Data.SqlClient.SqlCommandBuilder`) to create automatically the `INSERT`, `UPDATE`, and `DELETE` commands for you. In order to have conflict detection, add a **timestamp** column (alias **rowversion**) to the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] table that contains the UDT. The **timestamp** data type allows you to version-stamp the rows in a table, and is guaranteed to be unique within a database. When a value in the table is changed, [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] automatically updates the 8-byte binary number for the row affected by the change.

The `SqlCommandBuilder` doesn't consider the UDT for conflict detection unless there's a **timestamp** column in the underlying table. UDTs might or might not be comparable, so they aren't included in the `WHERE` clause when the "compare original values" option is used to generate a command.

### Example

The following example requires the creation of a second table containing the `Point` UDT column and a **timestamp** column. Both tables are used to illustrate how to create custom command objects to update data, and how to update using a **timestamp** column. Run the following [!INCLUDE [tsql](../../includes/tsql-md.md)] statements to create the second table and populate it with sample data.

```sql
CREATE TABLE dbo.Points_ts
(
    id INT PRIMARY KEY,
    p Point,
    ts TIMESTAMP
);

INSERT INTO dbo.Points_ts (id, p)
VALUES (1, CONVERT (Point, '1,3'));

INSERT INTO dbo.Points_ts (id, p)
VALUES (2, CONVERT (Point, '2,4'));

INSERT INTO dbo.Points_ts (id, p)
VALUES (3, CONVERT (Point, '3,5'));

INSERT INTO dbo.Points_ts (id, p)
VALUES (4, CONVERT (Point, '4,6'));
```

The following ADO.NET example has two methods:

- `UserProvidedCommands`, which demonstrates how to supply `InsertCommand`, `UpdateCommand`, and `DeleteCommand` objects for updating the `Point` UDT in the `Points` table (which doesn't contain a **timestamp** column).

- `CommandBuilder`, which demonstrates how to use a `SqlCommandBuilder` in the `Points_ts` table that contains the **timestamp** column.

### [C#](#tab/csharp)

```csharp
using System;
using System.Data;
using System.Data.SqlClient;

class Class1
{
// Retrieves the connection string
    private string connString = GetConnectionString();

    static void Main()
    {
        UserProvidedCommands();
        CommandBuilder();
    }

    static void UserProvidedCommands()
    {
        // Create a new SqlDataAdapter
        SqlDataAdapter da = new SqlDataAdapter(
            "SELECT id, p FROM dbo.Points", connString);

        // Setup the INSERT/UPDATE/DELETE commands
        SqlParameter idParam;
        SqlParameter pointParam;

        da.InsertCommand = new SqlCommand(
            "INSERT INTO dbo.Points (id, p) VALUES (@id, @p)",
            da.SelectCommand.Connection);
        idParam =
            da.InsertCommand.Parameters.Add("@id", SqlDbType.Int);
        idParam.SourceColumn = "id";
        pointParam =
            da.InsertCommand.Parameters.Add("@p", SqlDbType.Udt);
        pointParam.SourceColumn = "p";
        pointParam.UdtTypeName = "dbo.Point";

        da.UpdateCommand = new SqlCommand(
            "UPDATE dbo.Points SET p = @p WHERE id = @id",
            da.SelectCommand.Connection);
        idParam =
            da.UpdateCommand.Parameters.Add("@id", SqlDbType.Int);
        idParam.SourceColumn = "id";
        pointParam =
            da.UpdateCommand.Parameters.Add("@p", SqlDbType.Udt);
        pointParam.SourceColumn = "p";
        pointParam.UdtTypeName = "dbo.Point";

        da.DeleteCommand = new SqlCommand(
            "DELETE dbo.Points WHERE id = @id",
            da.SelectCommand.Connection);
        idParam =
            da.DeleteCommand.Parameters.Add("@id", SqlDbType.Int);
        idParam.SourceColumn = "id";

        // Fill the DataTable with UDT rows
        DataTable datTable = new DataTable("Points");
        da.Fill(datTable);

        // Display the contents of the p (Point) column
        foreach (DataRow r in datTable.Rows)
        {
            Point p = (Point)r[1];
            Console.WriteLine(
                "ID: {0}, x={1}, y={1}", r[0], p.X, p.Y);
        }

        // Update a row if the DataTable has at least 1 row
        if (datTable.Rows.Count > 0)
        {
            Point oldPoint = (Point)datTable.Rows[0][1];
            datTable.Rows[0][1] =
                new Point(oldPoint.X + 1, oldPoint.Y + 1);
        }

        // Delete the last row
        if (datTable.Rows.Count > 0)
        {
            // If we have at least 1 row
            datTable.Rows[1].Delete();
        }

        // Insert a row. This will fail if run twice
        // because 100 is a primary key value.
        datTable.Rows.Add(100, new Point(100, 200));

        // Send the changes back to the database
        da.Update(datTable);
    }

    static void CommandBuilder()
    {
        // Create a new SqlDataAdapter
        SqlDataAdapter da = new SqlDataAdapter(
            "SELECT id, ts, p FROM dbo.Points_ts", connString);

        // Select a few rows with UDTs from the database
        DataTable datTable = new DataTable("Points");
        da.Fill(datTable);

        // Display the contents of the p (Point) column
        foreach (DataRow r in datTable.Rows)
        {
            Point p = (Point)r[2];
            Console.WriteLine(
                "ID: {0}, x={1}, y={1}", r[0], p.X, p.Y);
        }

        // Update a row if DataTable has at least 1 row
        if (datTable.Rows.Count > 0)
        {
            Point oldPoint = (Point)datTable.Rows[0][2];
            datTable.Rows[0][2] =
                new Point(oldPoint.X + 1, oldPoint.Y + 1);
        }

        // Delete the last row
        if (datTable.Rows.Count > 0)
        {
            // if we have at least 1 row
            datTable.Rows[1].Delete();
        }

        // Insert a row. This will fail if run twice
        // because 100 is a primary key value
        datTable.Rows.Add(100, null, new Point(100, 200));

        // Use the CommandBuilder to generate DML statements
        SqlCommandBuilder bld = new SqlCommandBuilder(da);
        bld.ConflictDetection = ConflictOptions.CompareRowVersion;

        // Send the changes back to the database
        da.Update(datTable);
    }

    static private string GetConnectionString()
    {
        // To avoid storing the connection string in your code,
        // you can retrieve it from a configuration file.
        return "Data Source=localhost;Initial Catalog=AdventureWorks2022;"
               + "Integrated Security=SSPI";
    }
}
```

### [Visual Basic .NET](#tab/vb)

```vb
Imports System
Imports System.Data
Imports System.Data.SqlClient

Module Module1
    ' Retrieves the connection string
    Private connString As String = GetConnectionString()

    Sub Main()
        UserProvidedCommands()
        CommandBuilder()
    End Sub

    Private Sub UserProvidedCommands()
        ' Create a new SqlDataAdapter
        Dim da As New SqlDataAdapter( _
          "SELECT id, p FROM dbo.Points", connString)

        ' Setup the INSERT/UPDATE/DELETE commands
        Dim idParam As SqlParameter
        Dim pointParam As SqlParameter

        da.InsertCommand = New SqlCommand( _
          "INSERT INTO dbo.Points (id, p) VALUES (@id, @p)", _
          da.SelectCommand.Connection)
        idParam = da.InsertCommand.Parameters.Add( _
          "@id", SqlDbType.Int)
        idParam.SourceColumn = "id"
        pointParam = da.InsertCommand.Parameters.Add( _
          "@p", SqlDbType.Udt)
        pointParam.SourceColumn = "p"
        pointParam.UdtTypeName = "dbo.Point"

        da.UpdateCommand = New SqlCommand( _
          "UPDATE dbo.Points SET p = @p WHERE id = @id", _
          da.SelectCommand.Connection)
        idParam = _
           da.UpdateCommand.Parameters.Add("@id", SqlDbType.Int)
        idParam.SourceColumn = "id"
        pointParam = da.UpdateCommand.Parameters.Add( _
          "@p", SqlDbType.Udt)
        pointParam.SourceColumn = "p"
        pointParam.UdtTypeName = "dbo.Point"

        da.DeleteCommand = New SqlCommand( _
          "DELETE dbo.Points WHERE id = @id", _
          da.SelectCommand.Connection)
        idParam = da.DeleteCommand.Parameters.Add( _
          "@id", SqlDbType.Int)
        idParam.SourceColumn = "id"

        ' Fill the DataTable with UDT rows
        Dim datTable As New DataTable("Points")
        da.Fill(datTable)

        ' Display the contents of the p (Point) column
        Dim r As DataRow
        For Each r In datTable.Rows
            Dim p As Point = CType(r(1), Point)
            Console.WriteLine( _
              "ID: {0}, x={1}, y={1}", r(0), p.X, p.Y)
        Next r

        ' Update a row if the DataTable has at least 1 row
        If datTable.Rows.Count > 0 Then
            Dim oldPoint As Point = _
              CType(datTable.Rows(0)(1), Point)
            datTable.Rows(0)(1) = _
              New Point(oldPoint.X + 1, oldPoint.Y + 1)
        End If

        ' Delete the last row
        If datTable.Rows.Count > 0 Then
            ' If we have at least 1 row
            datTable.Rows(1).Delete()
        End If

        ' Insert a row. This will fail if run twice
        ' because 100 is a primary key value.
        datTable.Rows.Add(100, New Point(100, 200))

        ' Send the changes back to the database
        da.Update(datTable)
    End Sub

    Private Sub CommandBuilder()
        ' Create a new SqlDataAdapter
        Dim da As New SqlDataAdapter( _
          "SELECT id, ts, p FROM dbo.Points_ts", connString)

        ' Select a few rows with UDTs from the database
        Dim datTable As New DataTable("Points")
        da.Fill(datTable)

        ' Display the contents of the p (Point) column
        Dim r As DataRow
        For Each r In datTable.Rows
            Dim p As Point = CType(r(2), Point)
            Console.WriteLine( _
              "ID: {0}, x={1}, y={1}", r(0), p.X, p.Y)
        Next r

        ' Update a row if DataTable has at least 1 row
        If datTable.Rows.Count > 0 Then
            Dim oldPoint As Point = _
              CType(datTable.Rows(0)(2), Point)
            datTable.Rows(0)(2) = _
              New Point(oldPoint.X + 1, oldPoint.Y + 1)
        End If

        ' Delete the last row
        If datTable.Rows.Count > 0 Then
            ' if we have at least 1 row
            datTable.Rows(1).Delete()
        End If

        ' Insert a row. This will fail if run twice
        ' because 100 is a primary key value
        datTable.Rows.Add(100, Nothing, New Point(100, 200))

        ' Use the CommandBuilder to generate DML statements
        Dim bld As New SqlCommandBuilder(da)
        bld.ConflictDetection = ConflictOptions.CompareRowVersion

        ' Send the changes back to the database
        da.Update(datTable)
    End Sub

  Private Function GetConnectionString() As String
      ' To avoid storing the connection string in your code,
      ' you can retrieve it from a configuration file.
     Return "Data Source=(local);Initial Catalog=AdventureWorks2022;" _
       & "Integrated Security=SSPI"
   End Function
End Module
```

---

## Related content

- [Access user-defined types in ADO.NET](accessing-user-defined-types-in-ado-net.md)
