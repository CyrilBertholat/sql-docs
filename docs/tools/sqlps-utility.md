---
title: "sqlps Utility"
description: The sqlps utility starts a Windows PowerShell session with the SQL Server PowerShell provider and cmdlets loaded and registered.
author: markingmyname
ms.author: maghan
ms.date: "03/14/2017"
ms.service: sql
ms.subservice: tools-other
ms.topic: conceptual
helpviewer_keywords:
  - "sqlps utility"
  - "PowerShell [SQL Server], sqlps utility"
monikerRange: ">=aps-pdw-2016||=azuresqldb-current||=azure-sqldw-latest||>=sql-server-2016||>=sql-server-linux-2017"
---
# sqlps Utility
[!INCLUDE[SQL Server Azure SQL Database Synapse Analytics PDW](../includes/applies-to-version/sql-asdb-asdbmi-asa-pdw.md)]
  The **sqlps** utility starts a Windows PowerShell session with the [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] PowerShell provider and cmdlets loaded and registered. You can enter PowerShell commands or scripts that use the [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] PowerShell components to work with instances of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] and their objects.  
  
> [!IMPORTANT]  
>  [!INCLUDE[ssNoteDepFutureAvoid](../includes/ssnotedepfutureavoid-md.md)] Use the **sqlps** PowerShell module instead. For more information about the **sqlps** module, see [Import the SQLPS Module](/powershell/sql-server/download-sql-server-ps-module).  
  
## Syntax  
  
```  
  
sqlps   
[ [ [ -NoLogo ][ -NoExit ][ -NoProfile ]  
    [ -OutPutFormat { Text | XML } ] [ -InPutFormat { Text | XML } ]  
  ]  
  [ -Command { -  
             | script_block [ -args argument_array ]  
             | string [ command_parameters ]  
             }  
  ]  
]  
[ -? | -Help ]  
```  
  
## Arguments  
 **-NoLogo**  
 Specifies that the **sqlps** utility hide the copyright banner when it starts.  
  
 **-NoExit**  
 Specifies that the **sqlps** utility continue running after the startup commands have completed.  
  
 **-NoProfile**  
 Specifies that the **sqlps** utility not load a user profile. User profiles record commonly used aliases, functions, and variables for use across PowerShell sessions.  
  
 **-OutPutFormat** { **Text** | **XML** }  
 Specifies that the **sqlps** utility output be formatted as either text strings (**Text**) or in a serialized CLIXML format (**XML**).  
  
 **-InPutFormat** { **Text** | **XML** }  
 Specifies that input to the **sqlps** utility is formatted as either text strings (**Text**) or in a serialized CLIXML format (**XML**).  
  
 **-Command**  
 Specifies the command for the **sqlps** utility to run. The **sqlps** utility runs the command and then exits, unless **-NoExit** is also specified. Do not specify any other switches after **-Command**, they will be read as command parameters.  
  
 **-**  
 **-Command-** specifies that the **sqlps** utility read the input from the standard input.  
  
 *script_block* [ **-args**_argument\_array_ ]  
 Specifies a block of PowerShell commands to run, the block must be enclosed in braces: {}. *Script_block* can only be specified when the **sqlps** utility is called from either **PowerShell** or another **sqlps** utility session. The *argument_array* is an array of PowerShell variables containing the arguments for the PowerShell commands in the *script_block*.  
  
 *string* [ *command_parameters* ]  
 Specifies a string that contains the PowerShell commands to be run. Use the format **"&{**_command_**}"**. The quotation marks indicate a string, and the invoke operator (&) causes the **sqlps** utility to run the command.  
  
 [ **-?** | **-Help** ]  
 Shows the syntax summary of the **sqlps** utility options.  
  
## Remarks  
 The **sqlps** utility starts the PowerShell environment (PowerShell.exe) and loads the [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] PowerShell module. The module, also named **sqlps**, loads and registers these [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] PowerShell snap-ins:  
  
-   Microsoft.SqlServer.Management.PSProvider.dll  
  
     Implements the [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] PowerShell provider and associated cmdlets such as **Encode-SqlName** and **Decode-SqlName**.  
  
-   Microsoft.SqlServer.Management.PSSnapin.dll  
  
     Implements the **Invoke-Sqlcmd** and **Invoke-PolicyEvaluation** cmdlets.  
  
 You can use the **sqlps** utility to do the following:  
  
-   Interactively run PowerShell commands.  
  
-   Run PowerShell script files.  
  
-   Run [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] cmdlets.  
  
-   Use the [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] provider paths to navigate through the hierarchy of [!INCLUDE[ssNoVersion](../includes/ssnoversion-md.md)] objects.  
  
 By default, the **sqlps** utility runs with the scripting execution policy set to **Restricted**. This prevents running any PowerShell scripts. You can use the **Set-ExecutionPolicy** cmdlet to enable running signed scripts, or any scripts. Only run scripts from trusted sources, and secure all input and output files by using the appropriate NTFS permissions. For more information about enabling PowerShell scripts, see [Running Windows PowerShell Scripts](/previous-versions/system-center/virtual-machine-manager-2008-r2/cc917925(v=technet.10)).  
  
 The version of the **sqlps** utility in [!INCLUDE[sql2008-md](../includes/sql2008-md.md)] and [!INCLUDE[sql2008r2](../includes/sql2008r2-md.md)] was implemented as a Windows PowerShell 1.0 mini-shell. Mini-shells have certain restrictions, such as not allowing users to load snap-ins other than those loaded by the mini-shell. These restrictions do not apply to the [!INCLUDE[ssSQL11](../includes/sssql11-md.md)] and higher versions of the utility, which have been changed to use the **sqlps** module.  
  
## Examples  
 **A. Run the sqlps utility in default, interactive mode without the copyright banner**  
  
```  
sqlps -NoLogo  
```  
  
 **B. Run a SQL Server PowerShell script from the command prompt**  
  
```  
sqlps -Command "&{.\MyFolder.MyScript.ps1}"  
```  
  
 **C. Run a SQL Server PowerShell script from the command prompt, and keep running after the script completes**  
  
```  
sqlps -NoExit -Command "&{.\MyFolder.MyScript.ps1}"  
```  
  
## See Also  
 [Enable or Disable a Server Network Protocol](../database-engine/configure-windows/enable-or-disable-a-server-network-protocol.md)   
 [SQL Server PowerShell](/powershell/sql-server/sql-server-powershell)  
  
