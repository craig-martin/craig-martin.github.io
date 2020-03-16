---
layout: single
classes: wide
title:  "Adding Structured Data to Event Log Items"
share: false
tags: [PowerShell]
---
Ever want to store data while automating stuff? Often I need to store something but like to avoid writing files or introducing storage systems if I can avoid it, preferring to live off the land.  The scenario I'm working on right now is automating synchronization actions while providing status details.  Another scenario I'm working on is reading objects from one system and uploading them to Azure Log Analytics.  In the upload scenario I need to track the objects I've uploaded to avoid uploading the same object twice.

The solution I've come up with is pretty simple, write an event log entry to the Application event log.  This is pretty simple thanks to the ```Write-EventLog``` cmdlet:

```powershell
Write-EventLog
     [-LogName] 
     [-Source] 
     [[-EntryType] <EventLogEntryType>]
     [-Category <Int16>]
     [-EventId] 
     [-Message] 
     [-RawData <Byte[]>]
     [-ComputerName <String>]
     [<CommonParameters>]
```

Here is the one-line sample:

```powershell
# Write an entry to the Application log using MyCustomSource
Write-EventLog -LogName Application -Source MyCustomSource -Message foo -EventId 100
```

That works great (as long as you have already created the custom Source, see New-EventLog).  The fun comes when you want to use code to read the events, and get data from the events.  One approach is to stuff JSON into the event log item Message, similar to [what MIM Hybrid Reporting does](https://www.integrationtrench.com/2015/05/using-convertfrom-json-to-view-mim.html).

Another approach is to use the Message property for something human-readable (be kind to your event log readers) and use the RawData property for the structured data intended to be read by code.  Here is an example:

```powershell
# Create a custom event log
New-EventLog -LogName MyCustomLog -Source MyCustomSource

# Create a custom object, convert it to JSON
$json = [PSCustomObject]@{
    fooString = 'the foo'
    fooDate   = Get-Date
    fooArray  = 'foo','bar','baz'
    } | 
    ConvertTo-Json 

# Convert the JSON to a byte array
$jsonBytes = [System.Text.Encoding]::Unicode.GetBytes($json)

# Write an event log item with the raw data
Write-EventLog -LogName MyCustomLog -Source MyCustomSource -Message foo -EventId 100 -RawData $jsonBytes

# Get the event log item we just wrote
$event = Get-EventLog -LogName MyCustomLog -Newest 1

# Convert the RawData back to our object
# Notice how it preserves types (string, date, array)
[System.Text.Encoding]::Unicode.GetString($event.Data) | ConvertFrom-Json 

<#
fooString : the foo
fooDate   : @{value=1/25/2020 2:23:25 AM; DisplayHint=2; DateTime=Friday, January 24, 2020 6:23:25 PM}
fooArray  : {foo, bar, baz}
#>
```

There you have it, persisting structured data in the event log while maintaining something human readable.  It is more complex than the one-liner for creating a simple event log entry but it persists data without introducing a new storage mechanism, and the storage for the custom event log can be configured like any other event log. 