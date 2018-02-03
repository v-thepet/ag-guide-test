---
title: "Troubleshoot: Availability Group Exceeded RTO | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: e83e4ef8-92f0-406f-bd0b-dc48dc210517
caps.latest.revision: 7
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# Troubleshoot: Availability Group Exceeded RTO
  After an automatic failover or a planned manual failover without data loss on an availability group, you may find that the failover time exceeds your recovery time objective (RTO). Or, when you estimate the failover time of a synchronous-commit secondary replica (such as an automatic failover partner) using the method in [Monitor Performance for AlwaysOn Availability Groups](../ag-guide/monitor-performance-for-always-on-availability-groups.md), you find that it exceeds your RTO.  
  
 If your automatic failover still has not completed, see [Troubleshooting automatic failover problems in SQL Server 2012 AlwaysOn environments](http://support.microsoft.com/kb/2833707).  
  
## Common Causes  
 This section describes the common causes for a failover time that exceeds RTO.  
  
1.  [Reporting Workload Blocks the Redo Thread from Running](../ag-guide/troubleshoot-availability-group-exceeded-rto.md#BKMK_REDOBLOCK)  
  
2.  [Redo Thread Falls Behind Due to Resource Contention](../ag-guide/troubleshoot-availability-group-exceeded-rto.md#BKMK_CONTENTION)  
  
###  <a name="BKMK_REDOBLOCK"></a> Reporting Workload Blocks the Redo Thread from Running  
 The redo thread on the secondary replica is blocked from making data definition language (DDL) changes by a long-running read-only query.  
  
### Explanation  
 On the secondary replica, the read-only queries acquire schema stability (Sch-S) locks. These Sch-S locks can block the redo thread from acquiring schema modification (Sch-M) locks to make any DDL changes. A blocked redo thread cannot apply log records until it is unblocked. Once unblocked, it can continue to catch up to the end of log and allow the subsequent undo and failover process to proceed.  
  
### Diagnosis and Resolution  
 When the redo thread is blocked, an extended event called `sqlserver.lock_redo_blocked` is generated. Additionally, you can query the DMV sys.dm_exec_request on the secondary replica to find out which session is blocking the REDO thread, and then you can take corrective action. The following query returns the session ID of the read-only query that is blocking the redo thread.  
  
```tsql  
select session_id, command, blocking_session_id, wait_time, wait_type, wait_resource   
from sys.dm_exec_requests where command = 'DB STARTUP'  
```  
  
 You can let the reporting workload to finish, at which point the redo thread is unblocked. You can unblock the redo thread immediately by executing the [KILL &#40;Transact-SQL&#41;](../Topic/KILL%20(Transact-SQL).md) command on the blocking session ID.  
  
###  <a name="BKMK_CONTENTION"></a> Redo Thread Falls Behind Due to Resource Contention  
 A large reporting workload on the secondary replica has slowed down the performance of the secondary replica, and the redo thread has fallen behind.  
  
#### Explanation  
 When applying log records on the secondary replica, the redo thread reads the log records from the log disk, and then for each log record it accesses the data pages to apply the log record. The page access can be I/O bound (accessing the physical disk) if the page is not already in the buffer pool. If there is I/O bound reporting workload, the reporting workload competes for I/O resources with the redo thread and can slow down the redo thread.  
  
#### Diagnosis and Resolution  
 You can use the following DMV query to see how far the redo thread has fallen behind, by measuring the difference between the gap between `last_redone_lsn` and `last_received_lsn`.  
  
```tsql  
select recovery_lsn, truncation_lsn, last_hardened_lsn, last_received_lsn,   
   last_redone_lsn, last_redone_time  
from sys.dm_hadr_database_replica_states  
  
```  
  
 If the redo thread is indeed falling behind, you need to investigate the root cause of the performance degradation on the secondary replica. If there is an I/O contention with the reporting workload, you can use [Resource Governor](../Topic/Resource%20Governor.md) to control CPU cycles that are used by the reporting workload to indirectly control the I/O cycles taken, to some extent. For example, if your reporting workload is consuming 10 percent of CPU but the workload is I/O bound, you can use Resource Governor to limit CPU resource usage to 5 percent to throttle read workload, which minimizes the impact on I/O.  
  
## See Also  
 [Troubleshooting Performance Problems in SQL Server (Applies to SQL Server 2012)](http://msdn.microsoft.com/en-us/library/dd672789(v=SQL.100).aspx)  
  
  