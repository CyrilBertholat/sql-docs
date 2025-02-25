---
title: Configure peer-to-peer replication with one replica in an availability group
description: Describes how to configure a database in a SQL Server Always On availability group to participate in a peer-to-peer transactional replication topology.
author: MikeRayMSFT
ms.author: mikeray
ms.date: 09/25/2024
ms.service: sql
ms.topic: how-to
ms.custom:
  - template-how-to
  - updatefrequency5
---

# Configure one peer as part of availability group

Beginning with [!INCLUDE [sssql19-md](../../../../includes/sssql19-md.md)] CU 13 a database that belongs to a SQL Server Always On availability group can participate as a peer in a peer-to-peer transactional replication topology. This article describes how to configure this scenario.

The scripts in this example use T-SQL stored procedures.

## Roles & names

This section describes the roles and names of the various elements participating in the replication topology for this article.

### *Peer1*

- **Node1**: Replica 1 in the **MyAG** availability group.
- **Node2**: Replica 2 in the **MyAG** availability group.
- **MyAG**: The name of the availability group that you will create on *Node1* and *Node2*.
- **MyAGListenerName**: Name of availability group listener.
- **Dist1**: The name of the remote distributor instance.
- **MyDBName**: Name of the database.
- **P2P_MyDBName**: Publication name.

### *Peer2*

- **Node3**: A stand alone server hosting a default instance of SQL Server.
- **Dist2**: The name of the remote distributor instance.
- **MyDBName**: Name of the database.
- **P2P_MyDBName**: Publication name.

## Prerequisites

- Two instances of SQL Server on separate physical or virtual servers to host the availability group. The availability group will contain a peer database.
- One instance of SQL Server to host another peer database.
- Two instances of SQL Server to host the distributor databases.
- All server instances require a supported edition - Enterprise edition or Developer edition.
- All server instances require a supported version - [!INCLUDE [sssql19-md](../../../../includes/sssql19-md.md)] CU13 or later.
- Sufficient network connectivity and bandwidth between all instances.
- [Install SQL Server replication](../../../../database-engine/install-windows/install-sql-server-replication.md) on all instances of SQL Server.

   To see if replication is installed on any instance, run the following query:

   ```sql
   USE master;   
   GO   
   DECLARE @installed int;   
   EXEC @installed = sys.sp_MS_replication_installed;   
   SELECT @installed; 
   ```

  > [!NOTE]
  > To avoid a single point of failure for the distribution database, use a remote distributor for each peer.
  >
  > For demonstration or test environment, you may configure the distribution databases on a single instance.

## Configure the distributor and remote publisher (*Peer1*)

1. Run `sp_adddistributor` to configure distribution on *Dist1*. Use `@password =` to specify a password that the remote publisher uses to connect to the distributor. Use this password at each remote publisher when you configure the remote distributor.

   ```sql
   USE master;  
   GO  
   EXEC sys.sp_adddistributor  
    @distributor = 'Dist1',  
    @password = '<Strong password for distributor>';  
   ```

1. Create the distribution database at the distributor.

   ```sql
   USE master;
   GO  
   EXEC sys.sp_adddistributiondb  
    @database = 'distribution',  
    @security_mode = 1;  
   ```

1. Configure *Node1* and *Node2* as remote publisher.

   The `@security_mode` determines how the replication agents connect to the current primary.

      - `1` = Windows authentication.
      - `0` = SQL Server authentication. Requires `@login` and `@password`. The login and password specified must be valid at each secondary replica.
      
   > [!NOTE]
   > If any modified replication agents run on a computer other than the distributor, use of Windows authentication for the connection to the primary requires Kerberos authentication to  for the communication between the replica host computers. Use of a SQL Server login for the connection to the current primary doesn't require Kerberos authentication.

   ```sql
   USE master;  
   GO  
   EXEC sys.sp_adddistpublisher  
    @publisher = 'Node1',  
    @distribution_db = 'distribution',  
    @working_directory = '\\MyReplShare\WorkingDir',  
    @security_mode = 1
   USE master;  
   GO  
   EXEC sys.sp_adddistpublisher  
    @publisher = 'Node2',  
    @distribution_db = 'distribution',  
    @working_directory = '\\MyReplShare\WorkingDir',  
    @security_mode = 1
   ```

## Configure the publisher at the original publisher (*Node1*)

1. Configure remote distribution original publisher (*Node1*). Specify the same value for `@password` as that used when `sp_adddistributor` was run at the distributor to set up distribution.

   ```sql
   exec sys.sp_adddistributor  
   @distributor = 'Dist1',  
   @password = '<Password used when running sp_adddistributor on distributor server>' 
   ```

1. Enable the database for replication.

   ```sql
   USE master;  
   GO  
   EXEC sys.sp_replicationdboption  
    @dbname = 'MyDBName',  
    @optname = 'publish',  
    @value = 'true';  
   ```

## Configure the secondary replica host as replication publisher (*Node2*)

At each secondary replica host, configure distribution. Specify the same value for `@password` as that used when `sp_adddistributor` was run at the distributor to set up distribution.

```sql
EXEC sys.sp_adddistributor  
   @distributor = 'Dist1',  
   @password = '<Password used when running sp_adddistributor on distributor server>' 
```

## Make the database part of the availability group and create the listener (*Peer1*)

1. On the intended primary replica, create the availability group with the database as a member database.
1. Create a DNS listener for the availability group. The replication agent connects to the current primary replica by using the listener. The following example creates a listener named `MyAGListername`.

   ```sql
   ALTER AVAILABILITY GROUP 'MyAG'
   ADD LISTENER 'MyAGListenerName' (WITH IP (('<ip address>', '<subnet mask>') [, PORT = <listener_port>]));   
   ```

   > [!NOTE]
   > In the script above, information in square brackets (`[ ... ]`) is optional. Use it to specify a non default value for TCP port. Do not include the brackets.

## Redirect the original publisher to the AG listener name (*Peer1*)

On the distributor for *Peer1*, redirect the original publisher to the AG listener name.

```sql
USE distribution;   
GO   
EXEC sys.sp_redirect_publisher    
@original_publisher = 'Node1',   
@publisher_db = 'MyDBName',   
@redirected_publisher = 'MyAGListenerName,<port>';   
```

> [!NOTE]
> In the script above `,<port>` is optional. It is only required if you are using non-default ports. Do not include then angle brackets `<>`.

## Create peer-to-peer publication (*Peer1*) on the original publisher - *Node1*

The following script creates the publication for the Peer1.

```sql
exec master..sp_replicationdboption  @dbname=  'MyDBName'    
        ,@optname=  'publish'    
        ,@value=  'true'   

GO 

DECLARE @publisher_security_mode smallint = 1 
EXEC [MyDBName].dbo.sp_addlogreader_agent @publisher_security_mode = @publisher_security_mode 

GO 

DECLARE @allow_dts nvarchar(5) = N'false' 
DECLARE @allow_pull nvarchar(5) = N'true' 
DECLARE @allow_push nvarchar(5) = N'true' 
DECLARE @description nvarchar(255) = N'Peer-to-Peer publication of database MyDBName from Node1' 
DECLARE @enabled_for_p2p nvarchar(5) = N'true' 
DECLARE @independent_agent nvarchar(5) = N'true' 
DECLARE @p2p_conflictdetection nvarchar(5) = N'true' 
DECLARE @p2p_originator_id int = 100 
DECLARE @publication nvarchar(256) = N'P2P_MyDBName' 
DECLARE @repl_freq nvarchar(10) = N'continuous' 
DECLARE @restricted nvarchar(10) = N'false' 
DECLARE @status nvarchar(8) = N'active' 
DECLARE @sync_method nvarchar(40) = N'NATIVE' 
EXEC [MyDBName].dbo.sp_addpublication @allow_dts = @allow_dts, @allow_pull = @allow_pull, @allow_push = @allow_push, @description = @description, @enabled_for_p2p = @enabled_for_p2p,  
@independent_agent = @independent_agent, @p2p_conflictdetection = @p2p_conflictdetection, @p2p_originator_id = @p2p_originator_id, @publication = @publication, @repl_freq = @repl_freq, @restricted = @restricted,  
@status = @status, @sync_method = @sync_method 
go 

DECLARE @article nvarchar(256) = N'tbl0' 
DECLARE @description nvarchar(255) = N'Article for dbo.tbl0' 
DECLARE @destination_table nvarchar(256) = N'tbl0' 
DECLARE @publication nvarchar(256) = N'P2P_MyDBName' 
DECLARE @source_object nvarchar(256) = N'tbl0' 
DECLARE @source_owner nvarchar(256) = N'dbo' 
DECLARE @type nvarchar(256) = N'logbased' 
EXEC [MyDBName].dbo.sp_addarticle @article = @article, @description = @description, @destination_table = @destination_table, @publication = @publication, @source_object = @source_object, @source_owner = @source_owner, @type = @type 
go 

DECLARE @article nvarchar(256) = N'tbl1' 
DECLARE @description nvarchar(255) = N'Article for dbo.tbl1' 
DECLARE @destination_table nvarchar(256) = N'tbl1' 
DECLARE @publication nvarchar(256) = N'P2P_MyDBName' 
DECLARE @source_object nvarchar(256) = N'tbl1' 
DECLARE @source_owner nvarchar(256) = N'dbo' 
DECLARE @type nvarchar(256) = N'logbased' 
EXEC [MyDBName].dbo.sp_addarticle @article = @article, @description = @description, @destination_table = @destination_table, @publication = @publication, @source_object = @source_object,  
@source_owner = @source_owner, @type = @type 
GO  
```

## Make peer to peer publication compatible with availability group (*Peer1*)

On original publisher (Node1), run the following script to make the publication compatible with availability group:

```sql
USE MyDBName
GO
DECLARE @publication sysname = N'P2P_MyDBName' 
DECLARE @property sysname = N'redirected_publisher' 
DECLARE @value sysname = N'MyAGListenerName,<port>' 
EXEC MyDBName..sp_changepublication @publication = @publication, @property = @property, @value = @value 
GO 
```

> [!NOTE]
> In the script above `,<port>` is optional. It is only required if you are using non-default ports. Do not include then angle brackets `<>`.

After you have completed the steps above, the availability group is prepared to participate in peer-to-peer topology. The next steps configure a standalone instance of SQL Server (*Peer2*) to participate.

## Configure the distributor and remote publisher (*Peer2*)

1. Configure distribution at the distributor.

   ```sql
   USE master;   
   GO   
   EXEC sys.sp_adddistributor   
    @distributor = 'Dist2',   
    @password = '**Strong password for distributor**';   
   ```

1. Create the distribution database at the distributor.

   ```sql
   USE master;   
   GO   
   EXEC sys.sp_adddistributiondb   
    @database = 'distribution',   
    @security_mode = 1;     
   ```

1. Configure *Node3* as remote publisher on distributor *Dist2*

   ```sql
   USE master;   
   GO   
   EXEC sys.sp_adddistpublisher   
    @publisher = 'Node3',   
    @distribution_db = 'distribution',   
    @working_directory = '\\MyReplShare\WorkingDir2',   
    @security_mode = 1 
   ```

## Configure the publisher (*Peer2*)

1. On *Node3*, configure remote distribution. 

   ```sql
   exec sys.sp_adddistributor  
   @distributor = 'Dist2',  
   @password = '<Password used when running sp_adddistributor on distributor server>' 
   ```

1. On *Node3*, enable the database for replication.

```sql
USE master;  
GO  
EXEC sys.sp_replicationdboption  
    @dbname = 'MyDBName',  
    @optname = 'publish',  
    @value = 'true';  
```

## Create peer-to-peer publication (*Peer2*)

On *Node3* run the following command to create the peer-to-peer publication.

```sql
exec master..sp_replicationdboption  @dbname=  'MyDBName'   
        ,@optname=  'publish'   
        ,@value=  'true'  
go
DECLARE @publisher_security_mode smallint = 1
EXEC [MyDBName].dbo.sp_addlogreader_agent @publisher_security_mode = @publisher_security_mode
go

-- Note – Make sure that the value for @p2p_originator_id is different from Peer1.
DECLARE @allow_dts nvarchar(5) = N'false'
DECLARE @allow_pull nvarchar(5) = N'true'
DECLARE @allow_push nvarchar(5) = N'true'
DECLARE @description nvarchar(255) = N'Peer-to-Peer publication of database MyDBName from Node3'
DECLARE @enabled_for_p2p nvarchar(5) = N'true'
DECLARE @independent_agent nvarchar(5) = N'true'
DECLARE @p2p_conflictdetection nvarchar(5) = N'true'
DECLARE @p2p_originator_id int = 1
DECLARE @publication nvarchar(256) = N'P2P_MyDBName'
DECLARE @repl_freq nvarchar(10) = N'continuous'
DECLARE @restricted nvarchar(10) = N'false'
DECLARE @status nvarchar(8) = N'active'
DECLARE @sync_method nvarchar(40) = N'NATIVE'
EXEC [MyDBName].dbo.sp_addpublication @allow_dts = @allow_dts, @allow_pull = @allow_pull, @allow_push = @allow_push, @description = @description, @enabled_for_p2p = @enabled_for_p2p, @independent_agent = @independent_agent, @p2p_conflictdetection = @p2p_conflictdetection, @p2p_originator_id = @p2p_originator_id, @publication = @publication, @repl_freq = @repl_freq, @restricted = @restricted, @status = @status, @sync_method = @sync_method
GO

DECLARE @article nvarchar(256) = N'tbl0'
DECLARE @description nvarchar(255) = N'Article for dbo.tbl0'
DECLARE @destination_table nvarchar(256) = N'tbl0'
DECLARE @publication nvarchar(256) = N'P2P_MyDBName'
DECLARE @source_object nvarchar(256) = N'tbl0'
DECLARE @source_owner nvarchar(256) = N'dbo'
DECLARE @type nvarchar(256) = N'logbased'
EXEC [MyDBName].dbo.sp_addarticle @article = @article, @description = @description, @destination_table = @destination_table, @publication = @publication, @source_object = @source_object, @source_owner = @source_owner, @type = @type
GO

DECLARE @article nvarchar(256) = N'tbl1'
DECLARE @description nvarchar(255) = N'Article for dbo.tbl1'
DECLARE @destination_table nvarchar(256) = N'tbl1'
DECLARE @publication nvarchar(256) = N'P2P_MyDBName'
DECLARE @source_object nvarchar(256) = N'tbl1'
DECLARE @source_owner nvarchar(256) = N'dbo'
DECLARE @type nvarchar(256) = N'logbased'
EXEC [MyDBName].dbo.sp_addarticle @article = @article, @description = @description, @destination_table = @destination_table, @publication = @publication, @source_object = @source_object, 
@source_owner = @source_owner, @type = @type
GO
```

## Create a push subscription from *Peer1* to *Peer2*

This step creates a push subscription from the availability group to the standalone instance of SQL Server.

Execute the following script on *Node1*. This assumes *Node1* is running the primary replica.

```sql
EXEC [MyDBName].dbo.sp_addsubscription 
   @publication = N'P2P_MyDBName'
 , @subscriber = N'Node3'
 , @destination_db = N'MyDBName'
 , @subscription_type = N'push'
 , @sync_type = N'replication support only'
GO

EXEC [MyDBName].dbo.sp_addpushsubscription_agent 
   @publication = N'P2P_MyDBName'
 , @subscriber = N'Node3'
 , @subscriber_db = N'MyDBName'
 , @job_login = null
 , @job_password = null
 , @subscriber_security_mode = 1
 , @frequency_type = 64
 , @frequency_interval = 1
 , @frequency_relative_interval = 1
 , @frequency_recurrence_factor = 0
 , @frequency_subday = 4
 , @frequency_subday_interval = 5
 , @active_start_time_of_day = 0
 , @active_end_time_of_day = 235959
 , @active_start_date = 0
 , @active_end_date = 0
 , @dts_package_location = N'Distributor'
GO
```

## Create a push subscription from *Peer2* to the availability group listener

To create a push subscription from *Peer2* to the availability group listener, run the following command on *Node3*.

> [!IMPORTANT]
> The script below specifies the availability group listener name for the subscriber.
>
> ```sql
> @subscriber = N'MyAGListenerName,<port>'
> ```

```sql
EXEC [MyDBName].dbo.sp_addsubscription 
   @publication = N'P2P_MyDBName'
 , @subscriber = N'MyAGListenerName,<port>'
 , @destination_db = N'MyDBName'
 , @subscription_type = N'push'
 , @sync_type = N'replication support only'
GO

EXEC [MyDBName].dbo.sp_addpushsubscription_agent 
   @publication = N'P2P_MyDBName'
 , @subscriber = N'MyAGListenerName,<port>'
 , @subscriber_db = N'MyDBName'
 , @job_login = null
 , @job_password = null
 , @subscriber_security_mode = 1
 , @frequency_type = 64
 , @frequency_interval = 1
 , @frequency_relative_interval = 1
 , @frequency_recurrence_factor = 0
 , @frequency_subday = 4
 , @frequency_subday_interval = 5
 , @active_start_time_of_day = 0
 , @active_end_time_of_day = 235959
 , @active_start_date = 0
 , @active_end_date = 0
 , @dts_package_location = N'Distributor'
GO
```

## Configure linked servers

At each secondary replica host, make sure the push subscribers of publications appear as linked servers.

```sql
EXEC sys.sp_addlinkedserver   
    @server = 'MySubscriber';
```

## Next step

> [!div class="nextstepaction"]
> [Configure peer-to-peer publication database to be part of availability groups](multi-availability-group.md)
