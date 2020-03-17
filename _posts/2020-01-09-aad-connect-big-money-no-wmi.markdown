---
layout: single
classes: wide
share: false
title: AAD Connect - Big Money No WMI!
tags:
- PowerShell
- ADSync
- Azure
- WMI
- AAD
---

For years we've enjoyed access to sync functionality via WMI (pronounced '[Whammy](https://en.wikipedia.org/wiki/Press_Your_Luck)'), all the way back to MIIS. The [Windows Management Infrastructure provider for the sync engine](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/ms694615(v=vs.100)) provided some useful functionality, including:
* Start a management agent run or stop an existing management agent run.
* Get the results of a management agent run.
* Get statistical information about the connector space or metaverse.
* Set or change passwords.
* Get information about FIM Synchronization Service.
* Search for a connector space object with a specified domain and account name in a global address list, a Windows NT 4.0 domain, or in Active Directory Domain Services (AD DS).
* Search for a connector space object with a specified domain and user principal name in a global address list, a Windows NT 4.0 domain, or in AD DS.

The WMI provider lives on in MIM but goes away in AAD Connect starting in [version 1.4.18.0](https://docs.microsoft.com/en-us/azure/active-directory/hybrid/reference-connect-version-history#14180)

> Customers should be informed that the deprecated WMI endpoints for MIIS_Service have now been removed. Any WMI operations should now be done via PS cmdlets.

So heads up! If you have been programming against the WMI provider then you will need to refactor before upgrading to any version newer than 1.4.18.0 (ask me how I know). The ADSync PowerShell module does have a ton of cmdlets available, and I'll be sharing experiences with them in the coming posts.