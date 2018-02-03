---
title: "Troubleshoot: Changes on the Primary Replica are not Reflected on the Secondary Replica | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: c602fd39-db93-4717-8f3a-5a98b940f9cc
caps.latest.revision: 9
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# Troubleshoot: Changes on the Primary Replica are not Reflected on the Secondary Replica
  The client application completes an update on the primary replica successfully, but querying the secondary replica shows that the change is not reflected. This case assumes that your availability has a healthy synchronization state. In most cases, this behavior resolves itself after a few minutes.  
  
 If changes are still not reflected on the secondary replica after a few minutes, there may be a bottleneck in the synchronization work flow. The location of the bottleneck depends on whether the secondary replica is set to synchronous commit or asynchronous commit.  
  
 **Synchronous Commit**  
  
 Each successful update on the primary replica has already been synchronized to the secondary replica, or that the log records have already been flushed for hardening on the secondary replica. Therefore, the bottleneck should be in the redo process that happens after the log is flushed on the secondary replica.  
  
 However, once redo is caught up, all read workloads on the secondary replica are snapshot isolation  
  
-   Long-running transactions on the primary replica  
  
-   Redo on secondary replica  
  
 **Asynchronous Commit**  
  
 Since asynchronous commit acknowledges a transaction as soon as it is flushed to the local disk, the bottleneck can be anywhere after that point.  
  
-   Long-running transactions on the primary replica  
  
-   Network latency or throughput  
  
-   Log harden on the secondary replica  
  
-   Redo on the secondary replica  
  
## Common Causes  
 This section describes the common causes of changes on the primary replica not being reflected on the secondary replica for read-only queries.  
  
  
###  <a name="BKMK_OLDTRANS"></a> Long-Running Active Transactions  
 A long-running transaction on the primary replica prevents the updates from being read on the secondary replica.  
  
#### Explanation  
 All read workloads on the secondary replica are snapshot isolation queries. In snapshot isolation, read-only clients see the availability database on the secondary replica at the beginning point of the oldest active transaction in the redone log. If a transaction has not committed for hours, the open transaction blocks all read-only queries from seeing any new updates.  
  
#### Diagnosis and Resolution  
 On the primary replica, use [DBCC OPENTRAN &#40;Transact-SQL&#41;](../Topic/DBCC%20OPENTRAN%20(Transact-SQL).md) to view the oldest active transactions and see if they can be rolled back. Once, the oldest active transactions are rolled back and synchronized to the secondary replica, read workloads on the secondary replica can see updates in the availability database up to the beginning of the then-oldest active transaction.  
  
###  <a name="BKMK_LATENCY"></a> High Network Latency or Low Network Throughput Causes Log Build-up on the Primary Replica  
 High network latency or low throughput can prevent logs from being sent to the secondary replica fast enough.  
  
#### Explanation  
 The primary replica activates flow control on the log send when it has exceeded the maximum allowable number of unacknowledged messages sent over to the secondary replica. Until some of these messages have been acknowledged, no more log blocks can be sent to the secondary replica. This situation can have a more serious impact on potential data loss, possibly jeopardizing your recovery point objective (RPO).  
  
#### Diagnosis and Resolution  
 A high DMV value [log_send_queue_size](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) can indicate logs being held back at the primary replica. Dividing this value by [log_send_rate](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) can give you a rough estimate on how soon data can be caught up on the secondary replica.  
  
 Also, it is useful to check the two performance objects [SQL Server:Availability Replica > Flow Control Time (ms/sec)](../Topic/SQL%20Server,%20Availability%20Replica.md) and [SQL Server:Availability Replica > Flow control/sec](../Topic/SQL%20Server,%20Availability%20Replica.md). Multiplying these two values shows you in the last second how much time was spent waiting for flow control to clear. The longer the flow control wait time, the lower the send rate.  
  
 Below is a list of useful metrics in diagnosing network latency and throughput. You can use other Windows tools, such as **ping.exe** and [Resource Monitor](http://technet.microsoft.com/video/Video/ff710685) to evaluate network utilization.  
  
-   DMV [log_send_queue_size](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md)  
  
-   DMV [log_send_rate](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md)  
  
-   Performance counter **SQL Server:Database > Log Bytes Flushed/sec**  
  
-   Performance counter **SQL Server:Database Mirroring > Send/Receive Ack Time**  
  
-   Performance counter **SQL Server:Availability Replica > Bytes Sent to Replica/sec**  
  
-   Performance counter **SQL Server:Availability Replica > Bytes Sent to Transport/sec**  
  
-   Performance counter **SQL Server:Availability Replica > Flow Control Time (ms/sec)**  
  
-   Performance counter **SQL Server:Availability Replica > Flow Control/sec**  
  
-   Performance counter **SQL Server:Availability Replica > Resent Messages/sec**  
  
 To remedy this issue, try upgrading your network bandwidth or removing or reducing unnecessary network traffic.  
  
###  <a name="BKMK_REDOBLOCK"></a> Another Reporting Workload Blocks the Redo Thread from Running  
 The redo thread on the secondary replica is blocked from making data definition language (DDL) changes by a long-running read-only query. The redo thread must be unblocked before it can make further updates available for read workload.  
  
#### Explanation  
 On the secondary replica, the read-only queries acquire schema stability (Sch-S) locks. These Sch-S locks can block the redo thread from acquiring schema modification (Sch-M) locks to make any DDL changes. A blocked redo thread cannot apply log records until it is unblocked.  
  
#### Diagnosis and Resolution  
 When the redo thread is blocked, an extended event called `sqlserver.lock_redo_blocked` is generated. Additionally, you can query the DMV sys.dm_exec_request on the secondary replica to find out which session is blocking the REDO thread, and then you can take corrective action. The following query returns the session ID of the reporting workload that is blocking the redo thread.  
  
```tsql  
select session_id, command, blocking_session_id, wait_time, wait_type, wait_resource   
from sys.dm_exec_requests where command = 'DB STARTUP'  
```  
  
 You can let the reporting workload to finish, at which point the redo thread is unblocked. You can unblock the redo thread immediately by executing the [KILL &#40;Transact-SQL&#41;](../Topic/KILL%20(Transact-SQL).md) command on the blocking session ID.  
  
###  <a name="BKMK_REDOBEHIND"></a> Redo Thread Falls Behind Due to Resource Contention  
 A large reporting workload on the secondary replica has slowed down the performance of the secondary replica, and the redo thread has fallen behind.  
  
#### Explanation  
 When applying log records on the secondary replica, the redo thread reads the log records from the log disk, and then for each log record it accesses the data pages to apply the log record. The page access can be I/O bound (accessing the physical disk) if the page is not already in the buffer pool. If there is I/O bound reporting workload, the reporting workload competes for I/O resources with the redo thread and can slow down the redo thread. This situation not only affects other reporting workloads from seeing updated data, but it also affects RTO.  
  
#### Diagnosis and Resolution  
 You can use the following DMV query to see how far the redo thread has fallen behind, by measuring the difference between the gap between `last_redone_lsn` and `last_received_lsn`.  
  
```tsql  
select recovery_lsn, truncation_lsn, last_hardened_lsn, last_received_lsn,   
   last_redone_lsn, last_redone_time  
from sys.dm_hadr_database_replica_states  
  
```  
  
 If the redo thread is indeed falling behind, you need to investigate the root cause of the performance degradation on the secondary replica. If there is an I/O contention with the reporting workload, you can use [Resource Governor](../Topic/Resource%20Governor.md) to control CPU cycles that are used by the reporting workload to indirectly control the I/O cycles taken, to some extent. For example, if your reporting workload is consuming 10 percent of CPU but the workload is I/O bound, you can use Resource Governor to limit CPU resource usage to 5 percent to throttle read workload, which minimizes the impact on I/O.  
  
## See Also  
 [Troubleshooting Performance Problems in SQL Server 2008](https://msdn.microsoft.com/library/dd672789(v=sql.100).aspx)  
  
  