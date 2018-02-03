---
title: "Monitor Performance for AlwaysOn Availability Groups | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: dfd2b639-8fd4-4cb9-b134-768a3898f9e6
caps.latest.revision: 13
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# Monitor Performance for AlwaysOn Availability Groups
  The performance aspect of AlwaysOn Availability Groups is crucial to maintaining the service-level agreement (SLA) for your mission-critical databases. Understanding how AlwaysOn Availability Groups ship logs to secondary replicas can help you estimate the recovery time objective (RTO) and recovery point objective (RPO) of your AlwaysOn implementation and identify bottlenecks in poorly performing availability groups or replicas. This topic describes the synchronization process, shows you how to calculate some of the key metrics, and gives you the links to some of the common performance troubleshooting scenarios.  
  
 The following topics are covered:  
  
-   [Data Synchronization Process](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_DATA_SYNC_PROCESS)  
  
-   [Flow Control Gates](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_FLOW_CONTROL_GATES)  
  
-   [Estimating Failover Time (RTO)](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_RTO)  
  
-   [Estimating Potential Data Loss (RPO)](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_RPO)  
  
-   [Monitoring for RTO and RPO](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_Monitoring_for_RTO_and_RPO)  
  
-   [Performance Troubleshooting Scenarios](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_SCENARIOS)  
  
-   [Useful Extended Events](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_XEVENTS)  
  
##  <a name="BKMK_DATA_SYNC_PROCESS"></a> Data Synchronization Process  
 To estimate the time to full synchronization and to identify the bottleneck, you need to understand the synchronization process. Performance bottleneck can be anywhere in the process, and locating the bottleneck can help you dig deeper into the underlying issues. The figure and table below illustrate the data synchronization process:  
  
 ![AlwaysOn Availability Group Data Synchronization](../ag-guide/media/always-onag-datasynchronization.gif "AlwaysOn Availability Group Data Synchronization")  
  
|||||  
|-|-|-|-|  
|**Sequence**|**Step Description**|**Comments**|**Useful Metrics**|  
|1|Log Generation|Log data is flushed to disk. This log must be replicated to the secondary replicas. The log records enter the send queue.|[SQL Server:Database > Log bytes flushed\sec](../Topic/SQL%20Server,%20Databases%20Object.md)|  
|2|Capture|Logs for each database is captured and sent to the corresponding partner queue (one per database-replica pair). This capture process runs continuously as long as the availability replica is connected and data movement is not suspended for any reason, and the database-replica pair is shown to be either Synchronizing or Synchronized. If the capture process is not able to scan and enqueue the messages fast enough, the log send queue builds up.|[SQL Server:Availability Replica > Bytes Sent to Replica\sec](../Topic/SQL%20Server,%20Availability%20Replica.md), which is an aggregation of the sum of all database messages queued for that availability replica.<br /><br /> [log_send_queue_size](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) (KB) and [log_bytes_send_rate](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) (KB/sec) on the primary replica.|  
|3|Send|The messages in each database-replica queue is dequeued and sent across the wire to the respective secondary replica.|[SQL Server:Availability Replica > Bytes sent to transport\sec](../Topic/SQL%20Server,%20Availability%20Replica.md) and [SQL Server:Availability Replica > Message Acknowledgement Time](../Topic/SQL%20Server,%20Availability%20Replica.md) (ms)|  
|4|Receive and Cache|Each secondary replica receives and caches the message.|Performance counter [SQL Server:Availabiltiy Replica > Log Bytes Received/sec](../Topic/SQL%20Server,%20Availability%20Replica.md)|  
|5|Harden|Log is flushed on the secondary replica for hardening. After the log flush, an acknowledgement is sent back to the primary replica.<br /><br /> Once the log is hardened, data loss is avoided.|Performance counter [SQL Server:Database > Log Bytes Flushed/sec](../Topic/SQL%20Server,%20Databases%20Object.md)<br /><br /> Wait type [HADR_LOGCAPTURE_SYNC](../Topic/sys.dm_os_wait_stats%20(Transact-SQL).md)|  
|6|Redo|Redo the flushed pages on the secondary replica. Pages are kept in the redo queue as they wait to be redone.|[SQL Server:Database Replica > Redone Bytes/sec](../Topic/SQL%20Server,%20Database%20Replica.md)<br /><br /> [redo_queue_size](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) (KB) and [redo_rate](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md).<br /><br /> Wait type [REDO_SYNC](../Topic/sys.dm_os_wait_stats%20(Transact-SQL).md)|  
  
##  <a name="BKMK_FLOW_CONTROL_GATES"></a> Flow Control Gates  
 AlwaysOn Availability Groups is designed with flow control gates on the primary replica to avoid excessive resource consumption, such as network and memory resources, on all availability replicas. These flow control gates do not affect the synchronization health state of the availability replicas, but they can affect the overall performance of your availability databases, including RPO.  
  
 After the logs have been captured on the primary replica, they are subject to two levels of flow controls, as shown in the table below.  
  
|||||  
|-|-|-|-|  
|**Level**|**Number of Gates**|**Number of messages**|**Useful Metrics**|  
|Transport|1 per availabiltiy replica|8192|Extended event **database_transport_flow_control_action**|  
|Database|1 per availability database|11200 (x64)<br /><br /> 1600 (x86)|[DBMIRROR_SEND](../Topic/sys.dm_os_wait_stats%20(Transact-SQL).md)<br /><br /> Extended event **hadron_database_flow_control_action**|  
  
 Once the message threshold of either gate is reached, log messages are no longer sent to a specific replica or for a specific database. These can be sent once acknowledgement messages are received for the sent messages to bring the number of sent messages below the threshold.  
  
 In addition to the flow control gates, there is another factor that can prevent the log messages from being sent. The synchronization of replicas ensures that the messages are sent and applied in the order of the log sequence numbers (LSN). Before a log message is sent, its LSN also checked against the lowest acknowledged LSN number to make sure that it is less than one of thresholds (depending on the message type). If the gap between the two LSN numbers is larger than the threshold, the messages are not sent. Once the gap is below the threshold again, the messages are sent.  
  
 Two useful performance counters, [SQL Server:Availability Replica > Flow control/sec](../Topic/SQL%20Server,%20Availability%20Replica.md) and [SQL Server:Availability Replica > Flow Control Time (ms/sec)](../Topic/SQL%20Server,%20Availability%20Replica.md), show you, within the last second, how many times flow control was activated and how much time was spent waiting on flow control. Higher wait time on the flow control translate to higher RPO. For more information on the types of issues that can cause a high wait time on the flow control, see [Troubleshoot: Availability Group Exceeded RPO](../ag-guide/troubleshoot-availability-group-exceeded-rpo.md).  
  
##  <a name="BKMK_RTO"></a> Estimating Failover Time (RTO)  
 The RTO in your SLA depends on the failover time of your AlwaysOn implementation at any given time, which can be expressed in the following formula.  
  
 ![AlwaysOn Availability Groups RTO Calculation](../ag-guide/media/always-on-rto.gif "AlwaysOn Availability Groups RTO Calculation")  
  
> [!IMPORTANT]  
>  If an availability group contains more than one availability database, then the availability database with the highest Tfailover becomes the limiting value for RTO compliance.  
  
 The failure detection time, Tdetection, is the time it takes for the system to detect the failure. This time depends on cluster-level settings and not on the individual availability replicas. Depending on the automatic failover condition that is configured, a failover can be triggered as an instant response to a critical SQL Server internal error, such as orphaned spinlocks. In this case, detection can be as fast as the [sp_server_diagnostics &#40;Transact-SQL&#41;](../Topic/sp_server_diagnostics%20(Transact-SQL).md) error report is sent to the WSFC cluster (the default interval is 1/3 of the health check timeout). A failover can also be triggered because of a timeout, such as the cluster health check timeout has expired (30 seconds by default) or the lease between the resource DLL and the SQL Server instance has expired (20 seconds by default). In this case, detection time is as long as the timeout interval. For more information, see [Flexible Failover Policy for Automatic Failover of an Availability Group &#40;SQL Server&#41;](../Topic/Flexible%20Failover%20Policy%20for%20Automatic%20Failover%20of%20an%20Availability%20Group%20(SQL%20Server).md).  
  
 The only thing that the secondary replica needs to do to become ready for a failover is for the redo to catch up to the end of log. The redo time, Tredo, is calculated using the following formula:  
  
 ![AlwaysOn Availability Groups Redo Time Calculation](../ag-guide/media/always-on-redo.gif "AlwaysOn Availability Groups Redo Time Calculation")  
  
 where *redo_queue* is the value in [redo_queue_size](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) and *redo_rate* is the value in [redo_rate](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md).  
  
 The failover overhead time, Toverhead, includes the time it takes to fail over the WSFC cluster and to bring the databases online. This time is usually short and constant.  
  
##  <a name="BKMK_RPO"></a> Estimating Potential Data Loss (RPO)  
 The RPO in your SLA depends on the possible data loss of your AlwaysOn implementation at any given time. This possible data loss can be expressed in the following formula:  
  
 ![AlwaysOn Availability Groups RPO Calculation](../ag-guide/media/always-on-rpo.gif "AlwaysOn Availability Groups RPO Calculation")  
  
 where *log_send_queue* is the value of [log_send_queue_size](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md) and *log generation rate* is the value of [SQL Server:Database > Log Bytes Flushed/sec](../Topic/SQL%20Server,%20Databases%20Object.md).  
  
> [!WARNING]  
>  If an availability group contains more than one availability database, then the availability database with the highest Tdata_loss becomes the limiting value for RPO compliance.  
  
 The log send queue represents all the data that can be lost from a catastrophic failure. At first glance, it is curious that the log generation rate is used instead of the log send rate (see [log_send_rate](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md)). However, remember that using the log send rate only gives you the time to synchronize, while RPO measures data loss based on how fast it is generated, not on how fast it is synchronized.  
  
 A simpler way to estimate Tdata_loss is to use [last_commit_time](../Topic/sys.dm_hadr_database_replica_states%20(Transact-SQL).md). The DMV on the primary replica reports this value for all replicas. You can calculate the difference between the value for the primary replica and the value for the secondary replica to estimate how fast the log on the secondary replica is catching up to the primary replica. As stated previously, this calculation does not tell you the potential data loss based on how fast the log is generated, but it should be a close approximation.  
  
##  <a name="BKMK_Monitoring_for_RTO_and_RPO"></a> Monitoring for RTO and RPO  
 This section demonstrates how to monitor your availability groups for RTO and RPO metrics. This demonstration is similar to the GUI tutorial given in [The AlwaysOn Health Model Part 2 -- Extending the Health Model](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/13/extending-the-alwayson-health-model.aspx).  
  
 Elements of the failover time and potential data loss calculations in [Estimating Failover Time (RTO)](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_RTO) and [Estimating Potential Data Loss (RPO)](../ag-guide/monitor-performance-for-always-on-availability-groups.md#BKMK_RPO) are conveniently provided as performance metrics in the policy management facet **Database Replica State** (see [View the Policy-Based Management Facets on a SQL Server Object](../Topic/View%20the%20Policy-Based%20Management%20Facets%20on%20a%20SQL%20Server%20Object.md)). You can monitor these two metrics on a schedule and be alerted when the metrics exceed your RTO and RPO, respectively.  
  
 The demonstrated scripts create two system policies that are run on their respective schedules, with the following characteristics:  
  
-   An RTO policy that fails when estimated failover time exceeds 10 minutes, evaluated every 5 minutes  
  
-   An RPO policy that fails when estimated data loss exceeds 1 hour, evaluated every 30 minutes  
  
-   The two policies have identical configuration on all availability replicas  
  
-   Policies are evaluated on all servers, but only on the availability groups for which the local availability replica is the primary replica. If the local availability replica is not the primary replica, the policies are not evaluated.  
  
-   Policy failures are conveniently displayed in the AlwaysOn Dashboard when you view it on the primary replica.  
  
 Follow the instructions below on all server instances that participate in the availability group:  
  
1.  [Start the SQL Server Agent service](../Topic/Start,%20Stop,%20or%20Pause%20the%20SQL%20Server%20Agent%20Service.md) if it is not already started.  
  
2.  In SQL Server Management Studio, from the **Tools** menu, click **Options**.  
  
3.  In the **SQL Server AlwaysOn** tab, select **Enable user-defined AlwaysOn policy** and click **OK**.  
  
     This setting enables you to display properly configured custom policies in the AlwaysOn Dashboard.  
  
4.  Create a [Policy-Based Management condition](../Topic/Create%20a%20New%20Policy-Based%20Management%20Condition.md) using the following specifications:  
  
    -   **Name**: `RTO`  
  
    -   **Facet**: **Database Replica State**  
  
    -   **Field**: `Add(@EstimatedRecoveryTime, 60)`  
  
    -   **Operator**: **<=**  
  
    -   **Value**: `600`  
  
     This condition fails when potential failover time exceeds 10 minutes, including a 60 second overhead for both failure detection and failover.  
  
5.  Create a second [Policy-Based Management condition](../Topic/Create%20a%20New%20Policy-Based%20Management%20Condition.md) using the following specifications:  
  
    -   **Name**: `RPO`  
  
    -   **Facet**: **Database Replica State**  
  
    -   **Field**: `@EstimatedDataLoss`  
  
    -   **Operator**: **<=**  
  
    -   **Value**: `3600`  
  
     This condition fails when potential data loss exceeds 1 hour.  
  
6.  Create a third [Policy-Based Management condition](../Topic/Create%20a%20New%20Policy-Based%20Management%20Condition.md) using the following specifications:  
  
    -   **Name**: `IsPrimaryReplica`  
  
    -   **Facet**: **Availability Group**  
  
    -   **Field**: `@LocalReplicaRole`  
  
    -   **Operator**: **=**  
  
    -   **Value**: `Primary`  
  
     This condition checks whether the local availability replica for a given availability group is the primary replica.  
  
7.  Create a [Create a Policy-Based Management policy](../Topic/Create%20a%20Policy-Based%20Management%20Policy.md) using the following specifications:  
  
    -   **General** page:  
  
        -   **Name**: `CustomSecondaryDatabaseRTO`  
  
        -   **Check condition**: `RTO`  
  
        -   **Against targets**: **Every DatabaseReplicaState** in **IsPrimaryReplica AvailabilityGroup**  
  
             This setting ensures that the policy is evaluated only on availability groups for which the local availability replica is the primary replica.  
  
        -   **Evaluation mode**: **On schedule**  
  
        -   **Schedule**: **CollectorSchedule_Every_5min**  
  
        -   **Enabled**: **selected**  
  
    -   **Description** page:  
  
        -   **Category**: **Availability database warnings**  
  
             This setting enables the policy evaluation results to be displayed in the AlwaysOn Dashboard.  
  
        -   **Description**: **The current replica has an RTO that exceeds 10 minutes, assuming an overhead of 1 minute for discovery and failover. You should investigate performance issues on the respective server instance immediately.**  
  
        -   **Text to display**: **RTO Exceeded!**  
  
8.  Create a second [Create a Policy-Based Management policy](../Topic/Create%20a%20Policy-Based%20Management%20Policy.md) using the following specifications:  
  
    -   **General** page:  
  
        -   **Name**: `CustomAvailabilityDatabaseRPO`  
  
        -   **Check condition**: `RPO`  
  
        -   **Against targets**: **Every DatabaseReplicaState** in **IsPrimaryReplica AvailabilityGroup**  
  
        -   **Evaluation mode**: **On schedule**  
  
        -   **Schedule**: **CollectorSchedule_Every_30min**  
  
        -   **Enabled**: **selected**  
  
    -   **Description** page:  
  
        -   **Category**: **Availability database warnings**  
  
        -   **Description**: **The availability database has exceeded your RPO of 1 hour. You should investigate performance issues on the availability replicas immediately.**  
  
        -   **Text to display**: **RPO Exceeded!**  
  
 When you are done, two new SQL Server Agent jobs are created, one for each of the policy evaluation schedule. These jobs should have names that begin with **syspolicy_check_schedule**.  
  
 You can view the job history to inspect evaluation results. Evaluation failures will also be recorded in the Windows application log (in the Event Viewer) with the Event ID 34052. You can also configure SQL Server Agent to send alerts on the policy failures. For more information, see [Configure Alerts to Notify Policy Administrators of Policy Failures](../Topic/Configure%20Alerts%20to%20Notify%20Policy%20Administrators%20of%20Policy%20Failures.md).  
  
##  <a name="BKMK_SCENARIOS"></a> Performance Troubleshooting Scenarios  
 The table below lists the common performance-related troubleshooting scenarios.  
  
|Scenario|Description|  
|--------------|-----------------|  
|[Troubleshoot: Availability Group Exceeded RTO](../ag-guide/troubleshoot-availability-group-exceeded-rto.md)|After an automatic failover or a planned manual failover without data loss, the failover time exceeds your RTO. Or, when you estimate the failover time of a synchronous-commit secondary replica (such as an automatic failover partner), you find that it exceeds your RTO.|  
|[Troubleshoot: Availability Group Exceeded RPO](../ag-guide/troubleshoot-availability-group-exceeded-rpo.md)|After you perform a forced manual failover, your data loss is more than your RPO. Or, when you calculate the potential data loss of an asynchronous-commit secondary replica, you find that it exceeds your RPO.|  
|[Troubleshoot: Changes on the Primary Replica are not Reflected on the Secondary Replica](../ag-guide/troubleshoot-availability-primary-changes-not-reflected-on-secondary.md)|The client application completes an update on the primary replica successfully, but querying the secondary replica shows that the change is not reflected.|  
  
##  <a name="BKMK_XEVENTS"></a> Useful Extended Events  
 The following extended events are useful when troubleshooting replicas in the **Synchronizing** state.  
  
|Event Name|Category|Channel|Availability Replica|  
|----------------|--------------|-------------|--------------------------|  
|redo_caught_up|transactions|Debug|Secondary|  
|redo_worker_entry|transactions|Debug|Secondary|  
|hadr_transport_dump_message|alwayson|Debug|Primary|  
|hadr_worker_pool_task|alwayson|Debug|Primary|  
|hadr_dump_primary_progress|alwayson|Debug|Primary|  
|hadr_dump_log_progress|alwayson|Debug|Primary|  
|hadr_undo_of_redo_log_scan|alwayson|Analytic|Secondary|  
  
## See Also  
  
  
  