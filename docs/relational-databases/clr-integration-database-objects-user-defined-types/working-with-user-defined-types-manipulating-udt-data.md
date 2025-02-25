---
title: Manipulating UDT Data
description: This article describes how to insert, select, and update data in UDT columns of a SQL Server database.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
helpviewer_keywords:
  - "CAST function"
  - "data deletions [CLR integration]"
  - "data insertions [CLR integration]"
  - "CONVERT function"
  - "data updates [CLR integration]"
  - "comparing data"
  - "updating data [CLR integration]"
  - "removing data"
  - "IsByteOrdered property"
  - "variables [CLR integration]"
  - "data manipulation [CLR integration]"
  - "user-defined types [CLR integration], data manipulation"
  - "deleting data"
  - "UDTs [CLR integration], data manipulation"
  - "invoking UDT methods"
  - "inserting data"
dev_langs:
  - "TSQL"
---
# Manipulate user-defined type (UDT) data

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

[!INCLUDE [tsql](../../includes/tsql-md.md)] provides no specialized syntax for `INSERT`, `UPDATE`, or `DELETE` statements when modifying data in user-defined type (UDT) columns. The [!INCLUDE [tsql](../../includes/tsql-md.md)] `CAST` or `CONVERT` functions are used to cast native data types to the UDT type.

## Insert data in a UDT column

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statements insert three rows of sample data into the `Points` table. The `Point` data type consists of X and Y integer values that are exposed as properties of the UDT. You must use either the `CAST` or `CONVERT` function to cast the comma-delimited X and Y values to the `Point` type. The first two statements use the `CONVERT` function to convert a string value to the `Point` type, and the third statement uses the `CAST` function:

```sql
INSERT INTO dbo.Points (PointValue)
VALUES (CONVERT (Point, '3,4'));

INSERT INTO dbo.Points (PointValue)
VALUES (CONVERT (Point, '1,5'));

INSERT INTO dbo.Points (PointValue)
VALUES (CAST ('1,99' AS Point));
```

## Select data

The following `SELECT` statement selects the binary value of the UDT.

```sql
SELECT ID, PointValue
FROM dbo.Points;
```

To see the output displayed in a readable format, call the `ToString` method of the `Point` UDT, which converts the value to its string representation.

```sql
SELECT ID, PointValue.ToString() AS PointValue
FROM dbo.Points;
```

[!INCLUDE [ssresult-md](../../includes/ssresult-md.md)]

```output
ID PointValue
-- ----------
 1 3,4
 2 1,5
 3 1,99
```

You can also use the [!INCLUDE [tsql](../../includes/tsql-md.md)] `CAST` and `CONVERT` functions to achieve the same results.

```sql
SELECT ID, CAST (PointValue AS VARCHAR)
FROM dbo.Points;

SELECT ID, CONVERT (VARCHAR, PointValue)
FROM dbo.Points;
```

The `Point` UDT exposes its X and Y coordinates as properties, which you can then select individually. The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement selects the X and Y coordinates separately:

```sql
SELECT ID,
       PointValue.X AS xVal,
       PointValue.Y AS yVal
FROM dbo.Points;
```

The X and Y properties return an integer value, which is displayed in the result set.

```output
ID xVal yVal
-- ---- ----
 1    3    4
 2    1    5
 3    1   99
```

## Work with variables

You can work with variables using the `DECLARE` statement to assign a variable to a UDT type. The following statements assign a value using the [!INCLUDE [tsql](../../includes/tsql-md.md)] `SET` statement and display the results by calling the UDT's `ToString` method on the variable:

```sql
DECLARE @PointValue AS Point;

SET @PointValue = (SELECT PointValue
                   FROM dbo.Points
                   WHERE ID = 2);

SELECT @PointValue.ToString() AS PointValue;
```

The result set displays the variable value:

```output
PointValue
----------
-1,5
```

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statements achieve the same result using `SELECT` rather than `SET` for the variable assignment:

```sql
DECLARE @PointValue AS Point;

SELECT @PointValue = PointValue
FROM dbo.Points
WHERE ID = 2;

SELECT @PointValue.ToString() AS PointValue;
```

The difference between using `SELECT` and `SET` for variable assignment is that `SELECT` allows you to assign multiple variables in one `SELECT` statement, whereas the `SET` syntax requires each variable assignment to have its own `SET` statement.

## Compare data

You can use comparison operators to compare values in your UDT if you set the `IsByteOrdered` property to `true` when defining the class. For more information, see [Create user-defined types](creating-user-defined-types.md).

```sql
SELECT ID, PointValue.ToString() AS Points
FROM dbo.Points
WHERE PointValue > CONVERT (Point, '2,2');
```

You can compare internal values of the UDT regardless of the `IsByteOrdered` setting if the values themselves are comparable. The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement selects rows where X is greater than Y:

```sql
SELECT ID, PointValue.ToString() AS PointValue
FROM dbo.Points
WHERE PointValue.X < PointValue.Y;
```

You can also use comparison operators with variables, as shown in this query that searches for a matching `PointValue`.

```sql
DECLARE @ComparePoint AS Point;
SET @ComparePoint = CONVERT (Point, '3,4');

SELECT ID, PointValue.ToString() AS MatchingPoint
FROM dbo.Points
WHERE PointValue = @ComparePoint;
```

## Invoke UDT methods

You can also invoke methods that are defined in your UDT in [!INCLUDE [tsql](../../includes/tsql-md.md)]. The `Point` class contains three methods, `Distance`, `DistanceFrom`, and `DistanceFromXY`. For the code listings defining these three methods, see [Create user-defined types with ADO.NET](creating-user-defined-types-coding.md).

The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement calls the `PointValue.Distance` method:

```sql
SELECT ID,
       PointValue.X AS [Point.X],
       PointValue.Y AS [Point.Y],
       PointValue.Distance() AS DistanceFromZero
FROM dbo.Points;
```

The results are displayed in the `Distance` column:

```output
ID X  Y  Distance
-- -- -- ----------------
 1  3  4                5
 2  1  5 5.09901951359278
 3  1 99 99.0050503762308
```

The `DistanceFrom` method takes an argument of `Point` data type, and displays the distance from the specified point to the `PointValue`:

```sql
SELECT ID,
       PointValue.ToString() AS Pnt,
       PointValue.DistanceFrom(CONVERT (Point, '1,99')) AS DistanceFromPoint
FROM dbo.Points;
```

The results display the results of the `DistanceFrom` method for each row in the table:

```output
ID Pnt DistanceFromPoint
-- --- -----------------
 1 3,4  95.0210502993942
 2 1,5                94
 3 1,9                90
```

The `DistanceFromXY` method takes the points individually as arguments:

```sql
SELECT ID,
       PointValue.X AS X,
       PointValue.Y AS Y,
       PointValue.DistanceFromXY(1, 99) AS DistanceFromXY
FROM dbo.Points;
```

The result set is the same as the `DistanceFrom` method.

## Update data in a UDT column

To update data in a UDT column, use the [!INCLUDE [tsql](../../includes/tsql-md.md)] `UPDATE` statement. You can also use a method of the UDT to update the state of the object. The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement updates a single row in the table:

```sql
UPDATE dbo.Points
SET PointValue = CAST ('1,88' AS Point)
WHERE ID = 3;
```

You can also update UDT elements separately. The following [!INCLUDE [tsql](../../includes/tsql-md.md)] statement updates only the Y coordinate:

```sql
UPDATE dbo.Points
SET PointValue.Y = 99
WHERE ID = 3;
```

If the UDT is defined with byte ordering set to `true`, [!INCLUDE [tsql](../../includes/tsql-md.md)] can evaluate the UDT column in a `WHERE` clause.

```sql
UPDATE dbo.Points
SET PointValue = '4,5'
WHERE PointValue = '3,4';
```

### Update limitations

You can't update multiple properties at once using [!INCLUDE [tsql](../../includes/tsql-md.md)]. For example, the following `UPDATE` statement fails with an error because you can't use the same column name twice in one `UPDATE` statement.

```sql
UPDATE dbo.Points
SET PointValue.X = 5, PointValue.Y = 99
WHERE ID = 3;
```

To update each point individually, you would need to create a mutator method in the `Point` UDT assembly. You can then invoke the mutator method to update the object in a [!INCLUDE [tsql](../../includes/tsql-md.md)] `UPDATE` statement, as in the following example:

```sql
UPDATE dbo.Points
SET PointValue.SetXY(5, 99)
WHERE ID = 3;
```

## Delete data in a UDT column

To delete data in a UDT, use the [!INCLUDE [tsql](../../includes/tsql-md.md)] `DELETE` statement. The following statement deletes all rows in the table that match the criteria specified in the `WHERE` clause. If you omit the `WHERE` clause in a `DELETE` statement, all rows in the table are deleted.

```sql
DELETE dbo.Points
WHERE PointValue = CAST ('1,99' AS Point);
```

Use the `UPDATE` statement if you want to remove the values in a UDT column while leaving other row values intact. This example sets the `PointValue` to null.

```sql
UPDATE dbo.Points
SET PointValue = NULL
WHERE ID = 2;
```

## Related content

- [Work with user-defined types in SQL Server](working-with-user-defined-types-in-sql-server.md)
