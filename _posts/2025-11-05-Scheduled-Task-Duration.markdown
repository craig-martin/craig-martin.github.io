---
layout: single
classes: wide
share: false
title: How long do Scheduled Tasks take?
tags: [KQL]
---

For an ingestion pipeline I was using PowerShell, so needed a way to have a script run daily. 
The Hybrid Runbook Worker in an Azure Automation Account has been working really well with PowerShell 5, 
been relying on it for years, but this script needed PowerShell 7 because I've fallen in love with [Foreach-Object -Parallel](https://devblogs.microsoft.com/powershell/powershell-foreach-object-parallel-feature/).

The Hybrid Runbook Worker did not work well with PowerShell 7 so I decided to use Task Scheduler on a Windows Server instead (might as well, the server is long running and is already being paid for).

Since the script runs for a few hours, I want to observe the task duration as the script stabilizes.  If it takes too long then I have one issue, if it finishes too quickly then it is likely another issue.

The KQL below looks for two events from the computer:
- 100 = Task Started
- 102 = Task Completed

Event log entries have XML that can be parsed, and that is where the following details are found:
- Result - this key detail is used to determine the start and finish
- TaskName - not required for duration, but good to have, could have also used that instead of filtering for "My Task Name"
- InstanceId - the unique ID of the task run, could be used here but I didn't need it
- ActionName - not required for duration, but good to have

The event log entries look something like this:

| TimeGenerated	| TaskName	| InstanceId	| Result |
|--|--|--|--|
|2025-11-04T05:00	|My Task Name	|{6196321e-4a45-4a19-b552-d67b0f4fe11d}	|TaskStartEvent
|2025-11-04T09:17	|My Task Name	|{6196321e-4a45-4a19-b552-d67b0f4fe11d}	|TaskSuccessEvent
|2025-11-05T05:00	|My Task Name	|{f793df69-ba24-42b0-b695-03e094c4be87}	|TaskStartEvent
|2025-11-05T09:18	|My Task Name	|{f793df69-ba24-42b0-b695-03e094c4be87}	|TaskSuccessEvent

From there the job would be easy if the event log entry had data about the duration of the scheduled task.  
Unfortunately duration does not exist in the entry, but it can be calculated by comparing the start and finish time, which exists on two event log entries.

The star of the show is the [next function](https://learn.microsoft.com/en-us/kusto/query/next-function).  Next() will peek at a row ahead to get data.  
The query below uses Next() and produces data that looks like this:

|TimeGenerated|	TaskName|	InstanceId|	Result|	nextName|	nextTimestamp|	Minutes|
|--|--|--|--|--|--|--|
|2025-11-04T05:00|My Task Name|	{6196321e-4a45-4a19-b552-d67b0f4fe11d}|	TaskStartEvent|	TaskSuccessEvent|	2025-11-04T09:17|	4.2
|2025-11-05T05:00|My Task Name|	{f793df69-ba24-42b0-b695-03e094c4be87}|	TaskStartEvent|	TaskSuccessEvent|	2025-11-05T09:18|	4.3

Great!  The KQL query result now has a duration that can be used to make a pretty timechart!

``` kql
// How long do Scheduled Tasks take?
Event
| where TimeGenerated > ago(7d)
| where Source == 'Microsoft-Windows-TaskScheduler'
| where RenderedDescription contains "My Task Name"
| where EventID in (100, 102)  // 100 = Task Started, 102 = Task Completed
| extend  EventDataXml = parse_xml(EventData)
| extend 
    Result = EventDataXml.DataItem.EventData["@Name"],
    TaskName = EventDataXml.DataItem.EventData.Data[0]['#text'],
    InstanceId = EventDataXml.DataItem.EventData.Data[2]['#text'],
    ActionName = EventDataXml.DataItem.EventData.Data[2]['#text']
| order by TimeGenerated asc 
| project TimeGenerated, TaskName,InstanceId, Result
| extend nextName = next(Result), nextTimestamp = next(TimeGenerated)
| where Result == "TaskStartEvent" and nextName == "TaskSuccessEvent"
| extend Minutes = (nextTimestamp - TimeGenerated)/1hr
| project TimeGenerated, TaskName,InstanceId,Minutes
| project TimeGenerated, Minutes
| render timechart 
```
