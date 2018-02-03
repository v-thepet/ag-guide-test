---
title: "AlwaysOn Extended Events | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 5950f98a-3950-473d-95fd-cde3557b8fc2
caps.latest.revision: 6
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# AlwaysOn Extended Events
  SQL Server 2012 defines extended events that are specific to AlwaysOn. You can monitor these extended events in a session to help with root-cause diagnosis when you troubleshoot an availability group. You can view the AlwaysOn extended events using the following query:  
  
```tsql  
SELECT * FROM sys.dm_xe_objects WHERE name LIKE '%hadr%'  
```  
  
 [AlwaysOn_health Session](../ag-guide/always-on-extended-events.md#BKMK_AlwaysOn_health)  
  
 [Extended Events for Debugging](../ag-guide/always-on-extended-events.md#BKMK_Debugging)  
  
 [AlwaysOn Extended Events Reference](../ag-guide/always-on-extended-events.md#BKMK_Reference)  
  
##  <a name="BKMK_AlwaysOn_health"></a> AlwaysOn_health Session  
 The AlwaysOn_health extended events session is created automatically when you create the availability group and captures a subset of the AlwaysOn related events. This session is preconfigured as a useful and convenient tool to help you get started quickly while troubleshooting an availability group. The Create Availability Group Wizard automatically starts the session on every participating availability replica configured in the wizard.  
  
> [!IMPORTANT]  
>  If you did not create the availability group using the **New Availability Group Wizard**, the AlwaysOn_health session may not be automatically started. If the session is not started, it cannot capture event data when an unexpected issue occurs. You should manually start the session and configure the session to start automatically by configuring the session properties.  
  
 To view the definition of the AlwaysOn_health session:  
  
1.  In the **Object Explorer**, expand **Management**, **Extended Events**, and then **Sessions**.  
  
2.  Right-click **AlwaysOn_health**, then point to **Script Session as**, then point to **CREATE To**, and then click **New Query Editor Window**.  
  
 For information on a some of the events covered by AlwaysOn_health, see [AlwaysOn Extended Events Reference](../Topic/AlwaysOn%20Extended%20Events%20Reference.md).  
  
##  <a name="BKMK_Debugging"></a> Extended Events for Debugging  
 In addition to the extended events covered by the AlwaysOn_health session, SQL Server defines an extensive set of debug events for AlwaysOn Availability Groups. To harness these additional extended events in a session, follow the procedures below:  
  
1.  In the **Object Explorer**, expand **Management**, **Extended Events**, and then **Sessions**.  
  
2.  Right-click **Sessions** and select **New Session**. Or, right-click AlwaysOn_health and select **Properties**.  
  
3.  In the **Select a page** pane, click **Events**.  
  
4.  In the event library, in the **Category** column, select **alwayson** and clear all other categories.  
  
5.  In the **Channel** column, select **Debug**. All the AlwaysOn related events that are not already selected are now shown in the event library.  
  
6.  Highlight an event in the event library, then click the **>** button to select it for the session.  
  
7.  When finished with the session, click **OK** to close it. Make sure that the session is started so that it captures the events that you selected.  
  
##  <a name="BKMK_Reference"></a> AlwaysOn Extended Events Reference  
 This section describes some of the AlwaysOn extended events that are used by the [AlwaysOn Extended Events](../ag-guide/always-on-extended-events.md) to monitor the availability groups.  
  
 [availability_replica_state_change](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_availability_replica_state_change)  
  
 [availability_group_lease_expired](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_availability_group_lease_expired)  
  
 [availability_replica_automatic_failover_validation](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_availability_replica_automatic_failover_validation)  
  
 [error_reported (multiple error numbers): for transport or connection issues](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_error_reported)  
  
 [data_movement_suspend_resume](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_data_movement_suspend_resume)  
  
 [alwayson_ddl_executed](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_alwayson_ddl_executed)  
  
 [availability_replica_manager_state](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_availability_replica_manager_state)  
  
 [error_reported (1480): Database Replica Role Change](../Topic/AlwaysOn%20Extended%20Events%20Reference.md#BKMK_error_reported_1480)  
  
###  <a name="BKMK_availability_replica_state_change "></a> availability_replica_state_change  
 Occurs when the state of an availability replica has changed. The creation of an availability group or joining an availability replica can trigger this event. It is useful for the diagnostics of failed automatic failover. It can also be used to trace the failover steps.  
  
#### Event Information  
  
|Column|Description|  
|------------|-----------------|  
|Name|availability_replica_state_change|  
|Category|alwayson|  
|Channel|Operational|  
  
#### Event Fields  
  
|Name|Type_name|Description|  
|----------|----------------|-----------------|  
|availability_group_id|guid|The ID of the Availability Group.|  
|availability_group_name|unicode_string|The name of the Availability Group.|  
|availability_replica_id|guid|The ID of the Availability Replica.|  
|previous_state|availability_replica_state|The role of the replica before the change.<br /><br /> **Possible values are:**<br /><br /> Primary_Normal<br /><br /> Secondary_Normal<br /><br /> Resolving_Pending_Failover<br /><br /> Resolving_Normal<br /><br /> Primary_Pending<br /><br /> Not_Available|  
|current_state|availability_replica_state|The role of the replica after the change.<br /><br /> **Possible values are:**<br /><br /> Primary_Normal<br /><br /> Secondary_Normal<br /><br /> Resolving_Pending_Failover<br /><br /> Resolving_Normal<br /><br /> Primary_Pending<br /><br /> Not_Available|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
ADD EVENT availability_replcia_state_change  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
```  
  
###  <a name="BKMK_availability_group_lease_expired"></a> availability_group_lease_expired  
 Occurs when the cluster and availability group has a connectivity issue and the lease is expired. This event indicates that connectivity between the availability group and the underlying WSFC cluster is broken. If the connectivity issue occurs on the primary replica, the event may cause an automatic failover or cause the availability group to be offline.  
  
#### Event Information  
  
|Column|Description|  
|------------|-----------------|  
|Name|availability_group_lease_expired|  
|Category|alwayson|  
|Channel|Operational|  
  
#### Event Fields  
  
|Name|Type_name|Description|  
|----------|----------------|-----------------|  
|availability_group_id|guid|The id of availability group.|  
|availability_group_name|unicode_string|The name of availability group.|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
ADD EVENT availability_group_lease_expired  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
```  
  
###  <a name="BKMK_availability_replica_automatic_failover_validation"></a> availability_replica_automatic_failover_validation  
 Occurs when the automatic failover validates the readiness of an availability replica as a primary replica, and it shows whether the target availability replica is ready to be the new primary replica. For example, the failover validation returns false when not all databases are synchronized or not joined. This event is design to provide a failure point during failovers. This information is of interest to the database administrator especially for automatic failovers because an automatic failover is an unattended operation. The database administrator can review the event to see why an automatic failover failed.  
  
#### Event Information  
  
|Name|Description|  
|----------|-----------------|  
|availability_replica_automatic _failover_validation||  
|Category|alwayson|  
|Channel|Analytic|  
  
#### Event Fields  
  
|Name|Type_name|Description|  
|----------|----------------|-----------------|  
|availability_group_id|guid|The ID of the availability group.|  
|availability_group_name|unicode_string|The name of the availability group.|  
|availability_replica_id|guid|The ID of the availability replica.|  
|forced_quorum|validation_result_type|If the value is TRUE, then the automatic failover is invalidated on this availability replica.<br /><br /> TRUE<br /><br /> FALSE|  
|joined_and_synchronized|validation_result_type|If the value is FALSE, then the automatic failover is invalidated on this availability replica.<br /><br /> TRUE<br /><br /> FALSE|  
|previous_primary_or_automatic_failover_target|validation_result_type|If the value is FALSE, then the automatic failover is invalidated on this availability replica.<br /><br /> TRUE<br /><br /> FALSE|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
  
ADD EVENT availability_replica_automatic_failover_validation (  
WHERE (  
[forced_quorum]=(TRUE) OR [joined_and_synchronized]=(FALSE) OR [previous_primary_or_automatic_failover_target]=(TRUE)  
)  
)  
  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
  
```  
  
###  <a name="BKMK_error_reported"></a> error_reported (multiple error numbers): for transport or connection issues  
 Each filtered event indicates that a connectivity issue occurred in the transport or database mirroring endpoint that availability group depends on.  
  
|Column|Description|  
|------------|-----------------|  
|Name|error_reported<br /><br /> numbers to filter: 35201, 35202, 35206, 35204, 35207, 9642, 9666, 9691, 9692, 9693, 28034, 28036, 28080, 28091, 33309|  
|Category|errors|  
|Channel|Admin|  
  
#### Error Numbers to Filter  
  
|Error Number|Description|  
|------------------|-----------------|  
|35201|A connection timeout has occurred while attempting to establish a connection to availability replica '%ls'.|  
|35202|A connection for availability group '%ls' from availability replica '%ls' with id  [%ls] to '%ls' with id [%ls] has been successfully established.  This is an informational message only. No user action is required.|  
|35206|A connection timeout has occurred on a previously established connection to availability replica '%ls'.|  
|35204|Connection between instance '%ls' and '%ls' has been disabled due to endpoint shutdown.|  
|Timeout + connected|  
|35207|Connection attempt on availability group id '%ls' from replica id '%ls' to replica id '%ls' failed because of error %d severity %d state %d.  severity %d state %d. (this may not have a good DBA use. Check and remove later in that case)|  
|9642|An error occurred in a Service Broker/Database Mirroring transport connection endpoint  Error: %i  State: %i. (Near endpoint role: %S_MSG  far endpoint address: '%.*hs') Error: %i State: %i. (Near endpoint role: %S_MSG far endpoint address: '%.\*hs')|  
|9666|The %S_MSG endpoint is in disabled or stopped state.|  
|9691|The %S_MSG endpoint has stopped listening for connections.|  
|9692|The %S_MSG endpoint cannot listen on port %d because it is in use by another process.|  
|9693|The %S_MSG endpoint cannot listen for connections due to the following error: '%.*ls'.|  
|28034|Connection handshake failed. The login '%.*ls' does not have CONNECT permission on the endpoint. State %d.|  
|28036|Connection handshake failed. The certificate used by this endpoint was not found: %S_MSG. Use DBCC CHECKDB in master database to verify the metadata integrity of the endpoints. State %d.|  
|28080|Connection handshake failed. The %S_MSG endpoint is not configured. State %d.|  
|28091|Starting endpoint for %S_MSG with no authentication is not supported.|  
|33309|Cannot start cluster endpoint because the default %S_MSG endpoint  configuration has not been loaded yet.|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
ADD EVENT sqlserver.error_reported(  
    WHERE   
(  
--Connectivity Error Messages  
[error_number]=(35201)   
OR [error_number]=(35202)   
OR [error_number]=(35204)   
OR [error_number]=(35206)   
OR [error_number]=(35207)   
OR [error_number]=(9642)   
--OR [error_number]=(9666)   
OR [error_number]=(9691)   
OR [error_number]=(9692)   
OR [error_number]=(9693)   
OR [error_number]=(28034)   
OR [error_number]=(28036)   
OR [error_number]=(28080)   
OR [error_number]=(28091)   
OR [error_number]=(33309)  
)  
)  
  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
```  
  
###  <a name="BKMK_data_movement_suspend_resume"></a> data_movement_suspend_resume  
 Occurs when the database movement of a database replica is suspended or resumed.  
  
#### Event Information  
  
|Column|Description|  
|------------|-----------------|  
|Name|data_movement_suspend_resume|  
|Category|Alwayson|  
|Channel|Operational|  
  
#### Event Fields  
  
||||  
|-|-|-|  
|Name|Type_name|Description|  
|availability_group_id|guid|The id of the availability group.|  
|availability_group_name|unicode_string|The name of the availability group, if available.|  
|availability_replica_id|guid|The id of the availability replica.|  
|database_replica_id|guid|The id of the availability database.|  
|database_replica_name|unicode_string|The name of the availability database.|  
|database_id|uint32|The id of the availability database.|  
|suspend_status|suspend_status_type|The suspend status values.<br /><br /> SUSPEND_NULL<br /><br /> RESUMED<br /><br /> SUSPENDED<br /><br /> SUSPENDED_INVALID|  
|suspend_source|suspend_source_type|The source of the suspend or resume action.|  
|suspend_reason|unicode_string|The suspend reason captured in database replica manager.|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
  
ADD EVENT data_movement_suspend_resume (  
WHERE (  
[suspend_status]=(SUSPENDED)or [suspend_status]=(SUSPENDED_INVALID) or   
[suspend_status]=(SUSPEND_NULL)  
)  
)  
  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
```  
  
###  <a name="BKMK_alwayson_ddl_executed"></a> alwayson_ddl_executed  
 Occurs when an AlwaysOn data definition language (DDL) statement is being executed, including CREATE, ALTER or DROP. The main purpose of the event is to indicate an issue with a user action on an availability replica, or to indicate the starting point of an operational action, which is followed by a runtime issue such as a manual failover, a forced failover, suspended data movement, resumed data movement.  
  
#### Event Information  
  
|Column|Description|  
|------------|-----------------|  
|Name|alwayson_ddl_execution|  
|Category|alwayson|  
|Channel|Analytic|  
  
#### Event Fields  
  
|Name|Type_name|Description|  
|----------|----------------|-----------------|  
|availability_group_id|Guid|The ID of the availability group.|  
|availability_group_name|unicode_string|The name of the availability group.|  
|ddl_action|alwayson_ddl_action|Indicates the type of DDL action: CREATE, ALTER or DROP.|  
|ddl_phase|ddl_opcode|Indicates the phase of the DDL operation: BEGIN, COMMIT or ROLLBACK.|  
|Statement|unicode_string|The text of the statement that was executed.|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
  
ADD EVENT alwayson_ddl_executed  
  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
```  
  
###  <a name="BKMK_availability_replica_manager_state"></a> availability_replica_manager_state  
 Occurs when the state of the availability replica manager is changed. This event indicates the heartbeat of availability replica manager. When the availability replica manager is not in healthy state, all availability replicas in the SQL Server instance will be down.  
  
#### Event Information  
  
|Column|Description|  
|------------|-----------------|  
|Name|availability_replica_manager_state_change|  
|Category|alwayson|  
|Channel|Operational|  
  
#### Event Fields  
  
|Name|Type_name|Description|  
|----------|----------------|-----------------|  
|current_state|manager_state|The current state of the availability replica manager.<br /><br /> Online<br /><br /> Offline<br /><br /> WaitingForClusterCommunication|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
  
ADD EVENT availability_replica_manager_state (  
WHERE ([current_state]=(OFFLINE))  
)  
  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
```  
  
###  <a name="BKMK_error_reported_1480"></a> error_reported (1480): Database Replica Role Change  
 This filtered error_reported event occurs asynchronously after an availability replica role change. It indicates which availability database fails to change its expected role during the failover process.  
  
#### Event Information  
  
|Column|Description|  
|------------|-----------------|  
|Name|error_reported<br /><br /> Error Number 1480: The REPLICATION_TYPE_MSG database "DATABASE_NAME" is changing roles from “OLD_ROLE” to “NEW_ROLE” due to REASON_MSG|  
|Category|errors|  
|Channel|Admin|  
  
#### AlwaysOn_health Session Definition  
  
```tsql  
CREATE EVENT SESSION [AlwaysOn_health] ON SERVER   
ADD EVENT sqlserver.error_reported(  
    WHERE   
(  
--database replica role change message  
OR [error_number] = (1480)  
  
--database replica runtime error messages  
OR [error_number]=(823)   
OR [error_number]=(824)   
OR [error_number]=(829)  
)  
)  
  
ADD TARGET package0.event_file(SET filename=N'AlwaysOn_health.xel',max_file_size=(5),max_rollover_files=(4),metadatafile=N'AlwaysOn_health.xem')  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)  
GO  
```  
  
## See Also  
 [View Event Session Data](../Topic/View%20Event%20Session%20Data.md)   
 [AlwaysOn Extended Events Reference](../Topic/AlwaysOn%20Extended%20Events%20Reference.md)  
  
  