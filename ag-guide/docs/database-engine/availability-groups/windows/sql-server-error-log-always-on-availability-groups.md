---
title: "SQL Server Error Log (AlwaysOn Availability Groups) | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 39d0c98d-75af-4dd1-b908-30d31af56f2a
caps.latest.revision: 4
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# SQL Server Error Log (AlwaysOn Availability Groups)
  The SQL Server Error Log reports events affecting AlwaysOn Availability Groups, such as:  
  
-   Communication with the Windows Server Failover Clustering (WSFC) cluster  
  
-   State transitions of availability replicas  
  
-   State transitions of availability databases  
  
-   Connectivity state of availability databases between primary and secondary replicas  
  
-   Statuses of the availability group endpoints  
  
-   Statuses of the availability group listeners  
  
-   Lease status between the SQL Server resource DLL (running in the WSFC cluster) and the SQL Server instance (for more information, see [How It Works: SQL Server AlwaysOn Lease Timeout](http://blogs.msdn.com/b/psssql/archive/2012/09/07/how-it-works-sql-server-alwayson-lease-timeout.aspx))  
  
-   Error events in the availability group  
  
 The following symptoms should lead to review of the SQL Server Error Log:  
  
-   Cannot access availability databases  
  
-   Unexpected availability group failover  
  
-   Availability group is in the Resolving state unexpectedly  
  
-   Availability group is in an indeterminate state  
  
## See Also  
 [View the SQL Server Error Log &#40;SQL Server Management Studio&#41;](../Topic/View%20the%20SQL%20Server%20Error%20Log%20(SQL%20Server%20Management%20Studio).md)  
  
  