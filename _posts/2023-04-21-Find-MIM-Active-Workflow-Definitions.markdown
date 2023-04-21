---
layout: single
classes: wide
share: false
title: Find MIM Active Workflow Definitions 
tags: [MIM,PowerShell,Lithnet]
---

Need to find the workflow definitions that are actually configured to be active?  It's a good way to find configuration bloat that could be cleaned up.  

Query for WorkflowDefinition objects and use the ManagementPolicyRule reference attributes that point to the Action and Authorization workflows.

``` powershell
# Find workflow definitions used by enabled Management Policy Rules
Search-Resources -XPath "
    /WorkflowDefinition[
        ObjectID = /ManagementPolicyRule[Disabled='false']/ActionWorkflowDefinition 
        or 
        ObjectID = /ManagementPolicyRule[Disabled='false']/AuthorizationWorkflowDefinition
    ]" -AttributesToGet DisplayName | Select DisplayName
<#
DisplayName                                                       
-----------                                                       
Filter Validation Workflow for Administrators                     
Filter Validation Workflow for Non-Administrators                 
Group Expiration Notification Workflow                            
Group Validation Workflow                                         
Owner Approval Workflow         
#> 
```
