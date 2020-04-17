---
layout: single
classes: wide
title:  "Switch API preference for Log Alerts"
share: false
tags: [PowerShell, Azure, Log Analytics]
---

We use Azure Alert Rules quite a bit and have an old Log Analytics Workspace.  I read the article for how to [Switch API preference for Log Alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-log-api-switch#process-of-switching-from-legacy-log-alerts-api) and was excited because it promised:

> Your alert rules and monitoring are unaffected and the alerts will not stop or be stalled, during or after the switch. 

Unfortunately that was not the case when I made the switch, the process renamed all of our alert rules using some combination of GUIDs.  If you have an older Log Analytics workspace then proceed with caution until this issue is resolved:
[Changing API Preference results in Alert Rule names being overwritten with GUIDs #44936](https://github.com/MicrosoftDocs/azure-docs/issues/44936)
