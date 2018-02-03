---
title: "AlwaysOn Availability Groups Troubleshooting and Monitoring Guide | Microsoft Docs"
ms.custom: ""
ms.date: "2016-10-05"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "database-engine"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: 8d6d9954-ff6b-4e58-882e-eff0174f0d07
caps.latest.revision: 8
author: "rothja"
ms.author: "jroth"
manager: "jhubbard"
---
# AlwaysOn Availability Groups Troubleshooting and Monitoring Guide
  This guide helps you get started on troubleshooting some of the common issues in AlwaysOn Availability Groups and monitoring AlwaysOn Availability Groups. It is intended to provide original content as well as a landing page of useful information that is already published elsewhere.  
  
 While this guide cannot fully discuss all the issues that can occur on the large surface area covered by AlwaysOn Availability Groups, it can point you in the right direction in your root-cause analysis and resolution of the issues. As AlwaysOn Availability Groups is an integrated technology, many of the problems you encounter are only symptoms of other issues in your database system. Some issues are caused by settings within an availability group, such as an availability database being suspended. Other issues can include problems you can isolate to other aspects of SQL Server, such as SQL Server settings, database file deployments, and systemic performance issues unrelated to the availability group, replica, or database. Still other problems and exist outside of SQL Server, such as network I/O, TCP/IP, Active Directory, and Windows Server Failover Clustering (WSFC). Often, problems that surface in an availability group, replica, or database require you to troubleshoot multiple technologies before you can identify the root cause.  
  

  
##  <a name="BKMK_SCENARIOS"></a> Troubleshooting Scenarios  
 The table below contains links to the common troubleshooting scenarios for AlwaysOn Availability Groups. They are categorized by their scenario types, such as configuration, client connectivity, failover, and performance.  
  
|Scenario|Scenario Type|Description|  
|--------------|-------------------|-----------------|  
|[Troubleshoot AlwaysOn Availability Groups Configuration &#40;SQL Server&#41;](../Topic/Troubleshoot%20AlwaysOn%20Availability%20Groups%20Configuration%20(SQL%20Server).md)|Configuration|Provides information to help you troubleshoot typical problems with configuring server instances for AlwaysOn Availability Groups. Typical configuration problems include AlwaysOn Availability Groups is disabled, accounts are incorrectly configured, the database mirroring endpoint does not exist, the endpoint is inaccessible (SQL Server Error 1418), network access does not exist, and a join database command fails (SQL Server Error 35250).|  
|[Troubleshoot "Validating WSFC quorum vote configuration" warning](http://support.microsoft.com/kb/2833122)|Configuration|When you create an AlwaysOn availability group by using the New Availability Group Wizard in Microsoft SQL Server 2012, you receive a warning message that resembles the following: “The current WSFC cluster quorum vote configuration is not recommended for this availability group.”|  
|[Troubleshoot issues when creating availability group listeners](http://support.microsoft.com/kb/2829783)|Configuration|You encounter errors when trying to create an availability group listener.|  
|[Troubleshoot a Failed Add-File Operation &#40;AlwaysOn Availability Groups&#41;](../Topic/Troubleshoot%20a%20Failed%20Add-File%20Operation%20(AlwaysOn%20Availability%20Groups).md)|Configuration|An add-file operation caused the secondary database to be suspended and be in the NOT SYNCHRONIZING state.|  
|[Fix: Error 41009 when you try to create multiple availability groups](http://support.microsoft.com/kb/2711145/en-us)|Configuration|You encounter error 41009 when trying to create multiple availability groups.|  
|[Cannot connect to availability group listener in a multi-subnet environment](http://support.microsoft.com/kb/2792139/en-us)|Client Connectivity|After you configure the availability group listener, you are unable to ping the listener or connect to it from an application.|  
|[Troubleshoot failed automatic failovers](http://support.microsoft.com/kb/2833707)|Failover|An automatic failover did not complete successfully.|  
|[Troubleshoot: Availability Group Exceeded RTO](../../../../docs/database-engine/availability-groups/windows/troubleshoot-availability-group-exceeded-rto.md)|Performance|After an automatic failover or a planned manual failover without data loss, the failover time exceeds your RTO. Or, when you estimate the failover time of a synchronous-commit secondary replica (such as an automatic failover partner), you find that it exceeds your RTO.|  
|[Troubleshoot: Availability Group Exceeded RPO](../../../../docs/database-engine/availability-groups/windows/troubleshoot-availability-group-exceeded-rpo.md)|Performance|After you perform a forced manual failover, your data loss is more than your RPO. Or, when you calculate the potential data loss of an asynchronous-commit secondary replica, you find that it exceeds your RPO.|  
|[Troubleshoot: Changes on the Primary Replica are not Reflected on the Secondary Replica](../../../../docs/database-engine/availability-groups/windows/troubleshoot-availability-primary-changes-not-reflected-on-secondary.md)|Performance|The client application completes an update on the primary replica successfully, but querying the secondary replica shows that the change is not reflected.|  
  
##  <a name="BKMK_TOOLS"></a> Useful Tools for Troubleshooting  
 When configuring or running AlwaysOn Availability Groups, the different tools can help you diagnose different types of issues. The table below provides links to useful information on the tools.  
  
|Tool|Description|  
|----------|-----------------|  
|[Use the AlwaysOn Dashboard &#40;SQL Server Management Studio&#41;](../Topic/Use%20the%20AlwaysOn%20Dashboard%20(SQL%20Server%20Management%20Studio).md)|Reports an at-a-glance view of the health of your availability group in a user-friendly interface.|  
|[AlwaysOn Policies](../../../../docs/database-engine/availability-groups/windows/always-on-policies.md)|Used by the AlwaysOn Dashboard.|  
|[SQL Server Error Log &#40;AlwaysOn Availability Groups&#41;](../../../../docs/database-engine/availability-groups/windows/sql-server-error-log-always-on-availability-groups.md)|Logs state transition events for availability groups, replicas, and databases, statuses of other AlwaysOn components, and AlwaysOn errors.|  
|[CLUSTER.LOG &#40;AlwaysOn Availability Groups&#41;](../../../../docs/database-engine/availability-groups/windows/cluster-log-always-on-availability-groups.md)|Logs cluster events, including state transitions of the availability group resource, as well as events and errors from SQL Server resource DLL.|  
|[AlwaysOn Health Diagnostics Log](../../../../docs/database-engine/availability-groups/windows/always-on-health-diagnostics-log.md)|Logs SQL Server health diagnostics as reported to the WSFC cluster (SQL Server resource DLL) by [sp_server_diagnostics &#40;Transact-SQL&#41;](../Topic/sp_server_diagnostics%20(Transact-SQL).md).|  
|[Dynamic Management Views and System Catalog Views &#40;AlwaysOn Availability Groups&#41;](../../../../docs/database-engine/availability-groups/windows/dynamic-management-views-and-system-catalog-views-always-on-availability-groups.md)|Reports information on the availability groups such as configuration, health status, and performance metrics.|  
|[AlwaysOn Extended Events](../../../../docs/database-engine/availability-groups/windows/always-on-extended-events.md)|Provides detailed diagnotics of the availability groups and useful for root-cause analysis.|  
|[AlwaysOn Wait Types](../../../../docs/database-engine/availability-groups/windows/always-on-wait-types.md)|Provides wait statistics specific to availability groups and useful for performance tuning.|  
|AlwaysOn Performance Counters|Monitor AlwaysOn Availability Groups activity and are reflected in System Monitor, and is useful for performance tuning. For more information, see [SQL Server, Availability Replica](../Topic/SQL%20Server,%20Availability%20Replica.md) and [SQL Server, Database Replica](../Topic/SQL%20Server,%20Database%20Replica.md).|  
|[AlwaysOn Ring Buffers](../../../../docs/database-engine/availability-groups/windows/always-on-ring-buffers.md)|Record alerts within the SQL Server system for internal diagnostics, and can be used to debug issues related to the availability groups.|  
  
##  <a name="BKMK_MONITOR"></a> Monitoring AlwaysOn Availability Groups  
 The ideal time to troubleshoot an availability group is before a problem necessitates a failover, whether automatic or manual. This can be done by monitoring the availability group’s performance metrics and sending alerts when the availability replicas are performing outside the bounds of your service-level agreement (SLA). For example, if a synchronous secondary replica has performance issues that cause the estimated failover time to increase, you do not want to wait until an automatic failover occurs and you find out that the failover time exceeds your recovery time objective.  
  
 As AlwaysOn Availability Groups is a high availability and disaster recovery solution, the most important performance metrics to monitor are the estimated failover time, which affects your recovery time objective (RTO), and the potential data loss in a disaster, which affects your recovery point objective (RPO). You can gather these metrics from the data that SQL Server exposes at any given time, so you can be alerted of a problem in the HADR capabilities of your system before the actual failure events occur. Therefore, it is important to familiarize yourself with the data synchronization process of AlwaysOn Availability Groups and gather the metrics accordingly.  
  
 This table below points you to topics that can help you monitor the health of your AlwaysOn Availability Groups solution.  
  
|Topic|Description|  
|-----------|-----------------|  
|[Monitor Performance for AlwaysOn Availability Groups](../../../../docs/database-engine/availability-groups/windows/monitor-performance-for-always-on-availability-groups.md)|Describes the data synchronization process for AlwaysOn Availability Groups, the flow control gates, and useful metrics when monitoring an availability group; and also shows how to gather RTO and RPO metrics.|  
|[Monitoring of Availability Groups &#40;SQL Server&#41;](../Topic/Monitoring%20of%20Availability%20Groups%20(SQL%20Server).md)|Provides information on tools for monitoring an availability group.|  
|[The AlwaysOn Health Model Part 1 -- Health Model Architecture](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/09/overview-of-the-alwayson-manageability-health-model.aspx)|Provides an overview of the AlwaysOn health model.|  
|[The AlwaysOn Health Model Part 2 -- Extending the Health Model](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/13/extending-the-alwayson-health-model.aspx)|Shows how to customize the AlwaysOn health model and customize the AlwaysOn Dashboard to show extra information.|  
|[Monitoring AlwaysOn Health with PowerShell - Part 1: Basic Cmdlet Overview](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/13/monitoring-alwayson-health-with-powershell-part-1.aspx)|Provides a basic overview of the AlwaysOn PowerShell cmdlets that can be used to monitor the health of an availability group.|  
|[Monitoring AlwaysOn Health with PowerShell - Part 2: Advanced Cmdlet Usage](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/13/monitoring-alwayson-health-with-powershell-part-2.aspx)|Provides information on advanced usage of the AlwaysOn PowerShell cmdlets to monitor the health of an availability group.|  
|[Monitoring AlwaysOn Health with PowerShell - Part 3 : A Simple Monitoring Application](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/15/monitoring-alwayson-health-with-powershell-part-3.aspx)|Shows how to automatically monitor an availability group with an application.|  
|[Monitoring AlwaysOn Health with PowerShell - Part 4 : Integration with SQL Server Agent](http://blogs.msdn.com/b/sqlalwayson/archive/2012/02/15/the-always-on-health-model-part-4.aspx)|Provides information on how to integrate availability group monitoring with SQL Server Agent and configure notification to the appropriate parties when problems arise.|  
  
## See Also  
 [SQL Server AlwaysOn Team Blog](http://blogs.msdn.com/b/sqlalwayson/)   
 [CSS SQL Server Engineers Blogs](http://blogs.msdn.com/b/psssql/)  
  
  