---
title: "AlwaysOn Wait Types | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: afa8caff-f325-48d9-a8ef-a30beab60389
caps.latest.revision: 6
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# AlwaysOn Wait Types
  When trouble-shooting AlwaysOn latency, wait statistics can be monitored for accumulation using the AlwaysOn-specific wait types in the dynamic management view (DMV) [sys.dm_os_wait_stats &#40;Transact-SQL&#41;](../Topic/sys.dm_os_wait_stats%20(Transact-SQL).md).  
  
 For general information on using wait statistics, see [SQL Server 2005 Waits and Queues](https://technet.microsoft.com/library/cc966413.aspx). That document was written for SQL Server 2005, but its information can be applied to later SQL Server versions.  
  
## Query for AlwaysOn Wait Types  
 Use the T-SQL query below to retrieve all wait statistics with the AlwaysOn wait types:  
  
```tsql  
SELECT * FROM sys.dm_os_wait_stats   
WHERE wait_type LIKE '%hadr%'  
ORDER BY wait_time_ms DESC  
```  
  
 To monitor the wait statistics by capturing extended events, use the following T-SQL command.  
  
```  
CREATE EVENT SESSION [alwayson] ON SERVER   
ADD EVENT sqlos.wait_info(  
    WHERE ([wait_type]=(758) OR [wait_type]=(776) OR [wait_type]=(853) OR [wait_type]=(833)))  
WITH (MAX_MEMORY=4096 KB,EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS,MAX_DISPATCH_LATENCY=30 SECONDS,  
MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE,TRACK_CAUSALITY=OFF,STARTUP_STATE=OFF)  
GO  
```  
  
 You can view the key-value mapping of the wait type by running the following query:  
  
```  
SELECT * FROM sys.dm_xe_map_values   
WHERE name='wait_types' AND map_value LIKE '%hadr%'   
ORDER BY map_key ASC  
```  
  
## See Also  
 [Types of Waits](../Topic/sys.dm_os_wait_stats%20(Transact-SQL).md#WaitTypes)  
  
  