---
title: "Dynamic Management Views and System Catalog Views (AlwaysOn Availability Groups) | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 4320a4a4-6183-462b-8bda-e7424e7cb706
caps.latest.revision: 6
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# Dynamic Management Views and System Catalog Views (AlwaysOn Availability Groups)
  This topic shows you some of the common queries on the AlwaysOn dynamic management views (DMV) you can use to monitor and troubleshoot your availability groups.  
  
> [!TIP]  
>  In the AlwaysOn Dashboard, you can easily configure the GUI to display many of the DMVs for the availability replicas and availability databases by right-clicking the respective table header and selecting the DMV you wish to display or hide.  
  
 For more information on the AlwaysOn Availability Groups DMVs, see [AlwaysOn Availability Groups Dynamic Management Views and Functions &#40;Transact-SQL&#41;](../Topic/AlwaysOn%20Availability%20Groups%20Dynamic%20Management%20Views%20and%20Functions%20(Transact-SQL).md). For more information on the AlwaysOn Availability Groups catalog views, see [AlwaysOn Availability Groups Catalog Views &#40;Transact-SQL&#41;](../Topic/AlwaysOn%20Availability%20Groups%20Catalog%20Views%20(Transact-SQL).md).  
  
## Check the WSFC cluster node configuration  
 The following Transact-SQL (T-SQL) query retrieves the status of all the nodes in the current Windows Server Failover Clustering (WSFC) cluster.  
  
```tsql  
use master  
go  
select * from sys.dm_hadr_cluster_members  
go  
```  
  
 This result set reports the status of each member node of the current WSFC cluster. If the quorum is defined as **Node and File Share Majority**, even the file share is reported. You can see the status of each node, including the voting weight of each node (the [number_of_quorum_votes](../Topic/sys.dm_hadr_cluster_members%20(Transact-SQL).md) value).  
  
## Explore the cluster network  
 The following query retrieves the network configuration of the current WSFC cluster.  
  
```tsql  
select * from sys.dm_hadr_cluster_networks  
```  
  
 The result set contains one row for each network adapter in the WSFC cluster. For example, in a two-node cluster that contains two network adapters on each node, this query returns four rows.  
  
## Explore the availability groups  
 The following query retrieves information about an availability group.  
  
```tsql  
select primary_replica, primary_recovery_health_desc, synchronization_health_desc from sys.dm_hadr_availability_group_states  
go  
select * from sys.availability_groups  
go  
select * from sys.availability_groups_cluster  
go  
```  
  
 The DMVs [sys.dm_hadr_availability_group_states &#40;Transact-SQL&#41;](../Topic/sys.dm_hadr_availability_group_states%20(Transact-SQL).md), [sys.availability_groups &#40;Transact-SQL&#41;](../Topic/sys.availability_groups%20(Transact-SQL).md), and [sys.availability_groups_cluster](../Topic/sys.availability_groups_cluster%20(Transact-SQL).md) all return information about the availability groups in the current WSFC cluster. In fact, [sys.availability_groups &#40;Transact-SQL&#41;](../Topic/sys.availability_groups%20(Transact-SQL).md), and [sys.availability_groups_cluster](../Topic/sys.availability_groups_cluster%20(Transact-SQL).md) seem to return identical information.  
  
 However, [sys.availability_groups_cluster](../Topic/sys.availability_groups_cluster%20(Transact-SQL).md) reports availability group metadata stored in the WSFC Cluster, whereas [sys.availability_groups &#40;Transact-SQL&#41;](../Topic/sys.availability_groups%20(Transact-SQL).md) reports availability group metadata that is cached in the SQL Server process space. Furthermore, these two DMVs report configuration information whereas [sys.dm_hadr_availability_group_states &#40;Transact-SQL&#41;](../Topic/sys.dm_hadr_availability_group_states%20(Transact-SQL).md) reports the current health statuses of the availability groups.  
  
> [!IMPORTANT]  
>  This nomenclature carries forward with the DMVs that document availability replicas and availability databases.  
  
## Explore the availability replicas  
 The following query retrieves information about the availability replicas defined in your availability groups.  
  
```tsql  
select replica_id, role_desc, connected_state_desc, synchronization_health_desc from sys.dm_hadr_availability_replica_states  
go  
select replica_server_name, replica_id, availability_mode_desc, endpoint_url from sys.availability_replicas  
go  
select replica_server_name, join_state_desc from sys.dm_hadr_availability_replica_cluster_states  
go  
```  
  
 Similar to the availability group DMVs, you find three DMVs that report on availability replicas. [sys.dm_hadr_availability_replica_states](../Topic/sys.dm_hadr_availability_replica_states%20(Transact-SQL).md) reports state information about the availability replicas that is locally cached in SQL Server, and [sys.dm_hadr_availability_replica_cluster_states](../Topic/sys.dm_hadr_availability_replica_cluster_states%20(Transact-SQL).md) reports state information about the availability replicas from the WSFC cluster. Finally, [sys.availability_replicas](../Topic/sys.dm_hadr_availability_replica_cluster_states%20(Transact-SQL).md) reports configuration data on the availability replicas, which are cached locally in SQL Server.  
  
## Explore availability replica health  
 The following query retrieves current health information about the availability replicas.  
  
```tsql  
select replica_id, role_desc, recovery_health_desc, synchronization_health_desc from sys.dm_hadr_availability_replica_states  
go  
```  
  
 Compare the query results on the primary replica and on the secondary replica and note that on the secondary replica, health information is reported only for that replica and not for any other replica in the availability group.  
  
## Explore the availability databases  
 The following query retrieves information about the availability replicas defined in your availability group. You can observe the change in the query results before and after you suspend data movement on an availability database.  
  
```  
select * from sys.availability_databases_cluster  
go  
select group_database_id, database_name, is_failover_ready  from sys.dm_hadr_database_replica_cluster_states  
go  
select database_id, synchronization_state_desc, synchronization_health_desc, last_hardened_lsn, redo_queue_size, log_send_queue_size from sys.dm_hadr_database_replica_states  
go  
```  
  
 Here again, three AlwaysOn DMVs report on availability databases. [sys.availability_databases_cluster](../Topic/sys.availability_databases_cluster%20(Transact-SQL).md) reports configuration information about availability databases from the WSFC cluster. [sys.dm_hadr_database_replica_cluster_states](../Topic/sys.dm_hadr_database_replica_cluster_states%20(Transact-SQL).md) reports state information about the database replicas, which are locally cached in SQL Server. It contains some important state information, such as the availability replica’s failover readiness. Finally, [sys.dm_hadr_database_replica_states](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) is a very verbose result set which reports identity and state information on each availability database, such as LSN progress information for the logs of the primary and secondary database replicas.  
  
## Explore availability database health  
 The following query retrieves information about the health of each availability databases on the replicas. You can observe the change in the query results before and after you suspend data movement on an availability database.  
  
```tsql  
select dc.database_name, dr.database_id, dr.synchronization_state_desc,   
dr.suspend_reason_desc, dr.synchronization_health_desc  
from sys.dm_hadr_database_replica_states dr  join sys.availability_databases_cluster dc  
on dr.group_database_id=dc.group_database_id   
where is_local=1  
go  
```  
  
  