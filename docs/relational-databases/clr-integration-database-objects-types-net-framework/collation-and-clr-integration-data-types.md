---
title: "Collation and CLR Integration Data Types"
description: In SQL Server CLR integration, the .NET Framework string APIs use the CompareInfo property of CultureInfo of the current thread to perform string comparisons.
author: rwestMSFT
ms.author: randolphwest
ms.date: 12/27/2024
ms.service: sql
ms.subservice: clr
ms.topic: "reference"
helpviewer_keywords:
  - "data types [CLR integration]"
  - "parameter collation [CLR integration]"
  - "collations [CLR integration]"
---
# Collation and CLR integration data types

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

In the [!INCLUDE [dnprdnshort](../../includes/dnprdnshort-md.md)], the `CompareInfo` object handles collations. The [!INCLUDE [dnprdnshort](../../includes/dnprdnshort-md.md)] string application programming interfaces (APIs) use the `CompareInfo` property associated with the `CultureInfo` object of the current thread to perform string comparisons. The default setting of the `CultureInfo` object is based on the Windows locale setting for the computer on which [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] is running. This determines the default comparison semantics, if no explicit `CultureInfo` is specified, for comparisons of `System.String` values. [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] doesn't explicitly change the `CompareInfo` property to the database or server collation. If necessary, you must set the appropriate `CompareInfo` property in your routines.

## Parameter collation

When you create a common language runtime (CLR) routine, and a parameter of a CLR method bound to the routine is of type `SQLString`, [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] creates an instance of the parameter with the default collation of the database containing the calling routine. If a parameter isn't a `SqlType` (for example, `String` rather than `SQLString`), the collation information from the database isn't associated with the parameter.

## Related content

- [SQL Server data types in the .NET Framework](sql-server-data-types-in-the-net-framework.md)
