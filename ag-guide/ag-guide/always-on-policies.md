---
title: "AlwaysOn Policies | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 26bf8f71-c2b8-45ef-b3a3-372b96c9e6e3
caps.latest.revision: 7
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# AlwaysOn Policies
  The AlwaysOn Availability Groups system policies are used by the AlwaysOn Dashboard to provide information on the availability group health to the user. They are very useful for initial troubleshooting of an availability group’s operational issues. These policies can be extended and used to customize the AlwaysOn Dashboard, or run instantly to report the desired health information.  
  
 There are 14 system policies for AlwaysOn Availability Groups. For detailed information on each policy, see [AlwaysOn Policies for Operational Issues with AlwaysOn Availability Groups (SQL Server)](Always%20On%20Policies%20for%20Operational%20Issues%20with%20Always%20On%20Availability%20Groups%20\(SQL%20Server\).md).  
  
## View or Evaluate AlwaysOn Availability Groups System Policies  
 To view or run the AlwaysOn Availability Groups system policies in SQL Server Management Studio (SSMS), do the following:  
  
1.  In the **Object Explorer**, expand **Management**, **Policy Management**, **Policies**, and then **System Policies**.  
  
2.  Right-click one of the policies and click **Evaluate**. If you want to evaluate the policy you selected, you are done. You can click **View** in the **Target details** box to see the details of the evaluation results.  
  
3.  To view all the AlwaysOn Availability Groups system policies, in the **Select a page** pane, click **Policy Selection**.  
  
## See Also  
 [The AlwaysOn Health Model Part 2 -- Extending the Health Model](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/13/extending-the-alwayson-health-model.aspx)  
  
  