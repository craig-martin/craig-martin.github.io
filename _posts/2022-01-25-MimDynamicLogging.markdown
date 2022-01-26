---
layout: single
classes: wide
share: false
title: MIM Service Dyamic Logging
tags: [MIM]
---
# Dynamic Logging
Over the weekend a rather large MIM Service computer experienced an outage.  The FIMService service was taking 100% of the CPU, and was not responding to requests. 
Restarting the service seemed to mitigate the issue but the root cause remains elusive.  The event log did not have any good clues so I was going to increase logging for the MIM Service when I ran into this:
[MIM 2016 SP1 (4.4.1436.0) Service Dynamic Logging](https://docs.microsoft.com/en-us/microsoft-identity-manager/infrastructure/mim-service-dynamic-logging)

The article claims that dynamic logging allows MIM Service logging to be modified without restarting the MIM Service.  That does not appear to be the case on my test computer, but that's find.

There does seem to be some relationship between the Dynamic Logging level and the logging level of the items in the System.Diagnostics section of the configuration file.

Looking forward to turning the logging up to Information level (see [Trace Levels](https://docs.microsoft.com/en-us/dotnet/framework/wcf/diagnostics/tracing/configuring-tracing?redirectedfrom=MSDN#trace-level) then seeing just how much data gets produced. 

At the Information level the MIM Service provides some pretty useful detail for each request, including requests to the [Enumeration Endpoint](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/ee652436(v=vs.100)).  The MIM Request history has pretty useful detail for Create, Update and Delete operations but does not include Read detail.  
By collecting log data from the Enumeration Endpoint and uploading it to Log Analytics we will be able to get some valualbe insight into what the MIM API callers are doing.
