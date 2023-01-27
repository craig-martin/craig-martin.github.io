---
layout: single
classes: wide
share: false
title: Event Log Entry Count By Severity 
tags: [KQL,Windows]
---

Visualizing the count of Event Log events on a computer can be helpful in some cases:
- starting to investigate an issue - does the issue coincide with an increase in Event Log activity?
- deploying a change - did the change result in an increase in Event Log activity?

This is a very broad look, but can help get to the next questions, such as:
- What events are responsible for the increase?
- When did the increase occur?
- How does the increase relate to the issue?


``` 
// Count the Event Log Entries by Severity (Info, Warning, Error)
Event
| where TimeGenerated > ago(10d)
| where Computer == 'myComputerName'
| summarize count() by EventLevelName, bin(TimeGenerated, 1h)
| render timechart 
```

![image](https://user-images.githubusercontent.com/5515887/215154774-180d3d99-2b0f-4f44-898f-3431cfa0ce6f.png)

