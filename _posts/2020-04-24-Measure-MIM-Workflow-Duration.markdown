---
layout: single
classes: wide
title:  "Measure the Duration of a MIM Workflow"
share: false
tags: [MIM, Lithnet, PowerShell]
---

Investigating systems seems to be a game of asking questions then trying to get data to answer those questions.

Recently we experienced delays in notifications delivered by MIM, resulting in a poor user experience for notifications such as approvals that are time sensitive.

The question was simply: how long are notifcations taking to deliver?

The answer is simple if just looking at Request objects because Requests have attributes identifying when the start and end time:
``` powershell
Search-Resources -XPath @"
/Request[
    CreatedTime >= op:subtract-dayTimeDuration-from-dateTime(fn:current-dateTime(), xs:dayTimeDuration('PT5M')) 
    and 
    RequestStatus = 'Completed' 
]
"@ -AttributesToGet DisplayName, RequestStatus, CreatedTime, msidmCompletedTime
```

The Request duration however is not precise enough because a Request can contain multiple workflows, and some workflows such as approvals can require human intervention, creating an artificial answer to the question.

Requests have child objects of WorkflowInstance.  The WorkflowInstance objects track the running code in the workflow, for example the code required to deliver a notification.

WorkflowInstances do have a CreatedTime attribute, but there is no attribute identifying the end time.  The workaround is to just use the end time of the request; it's not precise but it's better.

The sample below shows how to use the [LithnetRMA](https://github.com/lithnet/resourcemanagement-powershell/wiki) to query for Request and WorkflowInstance objects, then to output the start and end time.

```powershell
$workflowDisplayName = 'Notify New Member with Welcome Message'

$workflowDurations = Search-Resources -XPath @"
/Request[
    CreatedTime >= op:subtract-dayTimeDuration-from-dateTime(fn:current-dateTime(), xs:dayTimeDuration('PT30M')) 
    and 
    RequestStatus = 'Completed' 
    and ActionWorkflowInstance = /WorkflowInstance[
        DisplayName='$workflowDisplayName']
    ]
"@ -AttributesToGet DisplayName, RequestStatus, msidmCompletedTime, ActionWorkflowInstance |
ForEach-Object {
    $completedTime = $PSItem.msidmCompletedTime
    $PSItem.ActionWorkflowInstance |
    ForEach-Object {
        $workflowInstance = Get-Resource -ID $PSItem -AttributesToGet CreatedTime, DisplayName
        [PSCustomObject]@{
            DisplayName   = $workflowInstance.DisplayName
            StartTime     = $workflowInstance.CreatedTime
            CompletedTime = $completedTime
            Duration      = $completedTime - $workflowInstance.CreatedTime
        }
    }
}
```

The script produces the following output:

```powershell
DisplayName                                 StartTime            CompletedTime        Duration        
-----------                                 ---------            -------------        --------        
Notify New Member with Welcome Message      4/24/2020 3:01:54 PM 4/24/2020 3:01:56 PM 00:00:02.1500000
Notify New Member with Welcome Message      4/24/2020 3:12:56 PM 4/24/2020 3:12:59 PM 00:00:03.6400000
Notify New Member with Welcome Message      4/24/2020 3:16:08 PM 4/24/2020 3:16:14 PM 00:00:05.1360000
Notify New Member with Welcome Message      4/24/2020 3:24:41 PM 4/24/2020 3:24:52 PM 00:00:10.0870000
Notify New Member with Welcome Message      4/24/2020 3:24:12 PM 4/24/2020 3:24:18 PM 00:00:05.5900000
Notify New Member with Welcome Message      4/24/2020 3:22:37 PM 4/24/2020 3:22:50 PM 00:00:13.0770000
Notify New Member with Welcome Message      4/24/2020 3:18:55 PM 4/24/2020 3:19:02 PM 00:00:06.3470000
Notify New Member with Welcome Message      4/24/2020 3:18:05 PM 4/24/2020 3:18:08 PM 00:00:02.6870000
Notify New Member with Welcome Message      4/24/2020 3:24:24 PM 4/24/2020 3:24:28 PM 00:00:04.9430000
```

The output shows that the WorkflowInstances for that workflow are finishing in a reasonable time.  When we had the issue the data was much worse ;-) and it really helped to know just how much worse.

BTW - most of these posts are intended for me, I re-use these snippets *all* the time.