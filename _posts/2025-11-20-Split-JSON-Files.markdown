---
layout: single
classes: wide
share: false
title: Splitting JSON Files for the Kusto Ingestion REST API Payload Limit
tags: [KQL,PowerShell]
---
Been using the [Kusto ingestion REST API](https://learn.microsoft.com/en-us/kusto/api/rest/streaming-ingest) recently and it's been really dependable, but ran into an issue with the JSON payload size.

The docs seem to say the limit is [more like 4GB](https://learn.microsoft.com/en-us/kusto/api/netfx/kusto-ingest-client-errors) but I think the ingestion REST API is different.

When calling the API with a JSON payload that is too large, the actual error returned is:

```powershell
# Upload a file to Kusto
$ingestUrl = "https://mycluster.pluto.kusto.windows.net/v1/rest/ingest/MyDatabase/MyTable`?streamFormat=MultiJSON&mappingName=MyMapping"
Invoke-WebRequest -Method POST -Uri $ingestUrl -InFile 'myGiantFile.json' -Headers @{Authorization= "Bearer $plainTextToken";'Content-Type'='application/json'} 

# Get the error
$responseStream = $error[0].Exception.Response.GetResponseStream()    
$responseStream.Position = 0
$sr = new-object System.IO.StreamReader $responseStream
$sr.ReadToEnd()
<#
Payload too large: Stream size is above size limit (10 Mb)
#>
```

To deal with the 10mb ingestion limit, I could either modify the extract to produce smaller JSON files, or leave the extract intact, and split the JSON files into smaller files.

For now I've chosen to split the files, since it will be reusable and keeps the extract free of the logic to create smaller files.

The PowerShell function to split JSON files is here:  

<script src="https://gist.github.com/craig-martin/de9a9b7d02763c008f1459329624ab37.js"></script>

It will need some improvements, which I plan to get to soon, but it works so far.

The approach is simple, but does not have great performance because it loads the entire JSON file into memory using Get-Content and ConvertFrom-Json.  Taking that approach got me to done faster, but I want to go back and improve the function using a forward-only reader to avoid loading the whole file into memory.  
At the moment I can get by with this approach because the input files are under 100MB, but I don't think this will scale to input files over 1GB.
