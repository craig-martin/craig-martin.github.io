---
layout: single
classes: wide
share: false
title: MIMDSC - Desired State Configuration Module for Microsoft Identity Manager
tags:
- PowerShell
- MIM
- DSC
---

Over the years I've automated deployments for MIM using a variety of methods but fell in love with Desired State Configuration enough to apply it to MIM. DSC was easy to love because at its heart it is state based, much like the synchronization engine has always been but DSC is focused on configuration while the synchronization is focused on identity. 

I decided to share some of the DSC resources on GitHub at the [MIMDSC repository](https://github.com/microsoft/mimdsc/wiki). The resources for MIM Sync are largely done.

To see what a synchronization configuration item looks like in DSC, check out the [examples in the wiki](https://github.com/Microsoft/MIMDSC/wiki/MimSyncExportAttributeFlowRule) such as:

```powershell
Configuration TestMimSyncExportAttributeFlowRule 
{ 
    Import-DscResource -ModuleName MimSyncDsc

    Node (hostname) 
    { 
        ExportAttributeFlowRule TestMimSyncExportAttributeFlowRule
        {
            ManagementAgentName    = 'TinyHR'
            MVObjectType           = 'SyncObject'
            CDAttribute            = 'JobTitle'
            CDObjectType           = 'person'
            Type                   = 'direct-mapping'
            SrcAttribute           = 'Title'
            SuppressDeletions      = $false
            Ensure                 = 'Present'
        }
    }
} 
```

I need to start the MIM Service resources but want to see if there is interest first.  If you happen to like both DSC and MIM Service configuration then let me know!