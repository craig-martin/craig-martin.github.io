---
layout: single
classes: wide
share: false
title: AAD Connect - ADSync Module Support for Remote PowerShell
tags:
- PowerShell
- ADSync
- AAD
---

Lately I've been having a blast replacing WMI calls with commands from the AAD Connect ADSync PowerShell module.  I hit an issue using the ADSync module with PowerShell remoting.  To illustrate:      
```powershell
Invoke-Command -ComputerName Sinking -ScriptBlock {Get-ADSyncRunProfileResult}
<#
There was no endpoint listening at net.pipe://localhost/ADSyncManagement that could accept the message. This is often caused by an incorrect address or SOAP 
action. See InnerException, if present, for more details.
#>
```

It seems some of the commands initiate a connection to an endpoint, which I suspect creates an authentication double-hop issue.<br /><br />If you are interested in helping prioritize this issue, please vote for the feedback item:
[ADSync Cmdlets Fail with Remote PowerShell](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/39400027-adsync-cmdlets-fail-with-remote-powershell)

The ADSync module has some great commands, help make them better with your votes :-)