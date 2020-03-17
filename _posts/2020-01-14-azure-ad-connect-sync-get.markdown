---
layout: single
classes: wide
share: false
title: 'Azure AD Connect sync: Get-ADSyncConnectorStatistics'
tags:
- PowerShell
- ADSync
- Azure
- WMI
- AAD
---

WMI gets deprecated in AAD Connect so I am working on updating some AAD Connect scripts to instead use the ADSync PowerShell module.&nbsp; The existing scripts prevent damage during synchronization by checking thresholds for:
* Import
  * number of pending import updates
  * number of pending import deletes
  * percentage of pending import updates
  * percentage of pending import deletes
* Export
  * number of pending export updates
  * number of pending export deletes
  * percentage of pending export updates
  * percentage of pending export deletes

In WMI the MicrosoftIdentityIntegrationServer:[MIIS_ManagementAgent](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/ms697764(v=vs.100)) class provided the import and export counters, as well as the total number of connector space objects.

In ADSync most of the counters are gone, the rest are moved:
```powershell
Get-ADSyncConnectorStatistics -ConnectorName litware.ca
<#
ExportAdds ExportUpdates ExportDeletes TotalConnectors
---------- ------------- ------------- ---------------
         0             0             0              94
#>
```

Unfortunately not enough detail to get feature parity with the scripts I'm working on because:
* Import counters are missing (can't prevent damage until export)
* 'TotalConnectors' is just that - the number of connectors in the connector space as opposed to the total number of objects in the connector space

My plan is to go with what ADSync provides for now, and implement the missing functionality later if it is important enough. I was a bit discouraged until I found [Azure AD Connect sync: Prevent accidental deletes](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-sync-feature-prevent-accidental-deletes), more on that later.