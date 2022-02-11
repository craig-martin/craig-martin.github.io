---
layout: single
classes: wide
share: false
title: Log Analytics Query Latency
tags: [Log Analytics,PowerShell]
---

Getting log data from systems into Log Analytics has transformed how I operate systems.  A good example is the MIM Synchronization Run History data, it contains critical details of synchronization operations but the user interface in MIM makes it very difficult to use.  To improve the situation, a couple years ago I started uploading Run History from synchronization servers to Log Analytics and it has been awesome.  This was my first time using the [Log Analytics Data Collector API](https://docs.microsoft.com/en-us/rest/api/loganalytics/create-request) and since then I've repeated the pattern with other interesting data including SQL index statistics, MIM Service request history, IIS SMTP logs, etc. 

Change tracking is a challenge for this approach because ideally a log item (Run History in this case) is only uploaded once.  In the Run History scenario it should be verified that the Run History (the unique combination of Run Number and Management Agent Name) are not already in Log Analytics.  When I first implemented this I used the Event Log on the synchronization server for change tracking; after each successful upload I would create an event in the Event Log using the Run Number as the Event ID, and using the Management Agent Name as the Event Description.  The Event Log is pretty dependable storage, so this worked great until the Run Number exceeded the limit of the Event ID property (oops), so I needed a better method of change tracking.

My new approach to change tracking is to use the [Log Analytics Query API](https://docs.microsoft.com/en-us/rest/api/loganalytics/dataaccess/query) which works great but has a slight challenge, which is the topic of this post.

Log Analytics upload and query APIs are quite different, more cousins than siblings:
* different API endpoints
* different authentication (API keys versus AAD Access Tokens)
* different versions

The issue to be aware of is the query latency after uploading data.  When uploading data to a SQL database you can expect the data to be available to queries immediately.  In this case Log Analytics behaves more like Active Directory replication where a write operation performed on one domain controller will not be immediately available from a different domain controller until that change has been replicated.

To get a feel for how long it took between the data being written to when the data could be read, I put this small script together.  The goal was to get a feel for how much latency there actually was, to determine if the Log Analytics Query API could be used for change tracking.  The results from some random testing shows that data can be read in a small number of minutes (under five).  Since I am only uploading data hourly it is acceptable to have latency of a few minutes.

The script below uses both the Data Collector API and the Query API.  The script ran on an Azure VM so I was able to use the VM's identity to get an AAD access token (I love how that works).

``` powershell
$logAnalyticsWorkspaceID  = 'foo'
$logAnalyticsWorkspaceKey = 'bar'

<#
Upload a test string to Log Analytics using the Data Collector API
https://docs.microsoft.com/en-us/rest/api/loganalytics/create-request
#>
$LogAnalyticsDataCollectorApi = "https://" + $logAnalyticsWorkspaceID + ".ods.opinsights.azure.com/api/logs?api-version=2016-04-01"

$testStringData = "bar $(Get-Random)"
$JsonBody = "[{'foo': '$testStringData'}]" 

$stringToHash = "POST`n$($JsonBody.Length)`napplication/json`nx-ms-date:$([DateTime]::UtcNow.ToString("r"))`n/api/logs"

$bytesToHash = [Text.Encoding]::UTF8.GetBytes($stringToHash)
$keyBytes = [Convert]::FromBase64String($logAnalyticsWorkspaceKey)

$sha256 = New-Object System.Security.Cryptography.HMACSHA256
$sha256.Key = $keyBytes
$calculatedHash = $sha256.ComputeHash($bytesToHash)
$encodedHash = [Convert]::ToBase64String($calculatedHash)

$headers = @{
    "Authorization"        = 'SharedKey {0}:{1}' -f $logAnalyticsWorkspaceID, $encodedHash
    "Log-Type"             = 'FooLog'
    "x-ms-date"            = [DateTime]::UtcNow.ToString("r")
}
$response = Invoke-WebRequest -Uri $LogAnalyticsDataCollectorApi -Method Post -ContentType 'application/json' -Headers $headers -Body $JsonBody -UseBasicParsing
$response.StatusDescription


<#
Get the test data from Log Analytics using the Query API
  In this case the VM running the script has been granted Log Analytics Reader permission to the Log Analytics Workspace
https://docs.microsoft.com/en-us/rest/api/loganalytics/dataaccess/query
#>
$LogAnalyticsQueryApi = 'https://westus2.api.loganalytics.io'  
Write-Verbose "[$(Get-Date) $(hostname)] Getting AAD access token for:       $LogAnalyticsBaseURI"
$AzureInstanceMetadataServiceUrl = 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01' 
try{
	$token = Invoke-RestMethod -Uri "$AzureInstanceMetadataServiceUrl&resource=$LogAnalyticsBaseURI" -Headers @{Metadata=$true} -Verbose:$false
}
catch{
	throw "Failed to get AAD access token for resource: $LogAnalyticsBaseURI : $_"
}

$start = Get-Date
1..1000 | ForEach-Object{    
    Write-Host "[$(Get-Date)] Query Log Analytics for: $testStringData " -NoNewline
    $logQuery = "FooLog_CL | where foo_s == '$testStringData'"
    $logQueryBody = @{"query" = $logQuery} | ConvertTo-Json
    $queryResult = Invoke-RestMethod -Method POST -Uri "https://westus2.api.loganalytics.io/v1/workspaces/$logAnalyticsWorkspaceID/query" -Headers @{Authorization = "Bearer $($token.access_token)"; "Content-Type" = "application/json"} -Body $logQueryBody   
    if($queryResult.tables[0].rows)
    {
        Write-Host "  Got it in $([Math]::Round(((Get-Date) - $start).TotalMinutes,2)) minutes" -ForegroundColor Green
        break
    }
    Write-Host "  Nope. Sleeping..." -ForeGroundColor Yellow
    Start-Sleep -Seconds 10
}

<#
[02/11/2022 11:15:27] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:15:38] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:15:48] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:15:58] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:16:08] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:16:18] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:16:28] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:16:38] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:16:48] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:16:58] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:17:08] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:17:18] Query Log Analytics for: bar 1520676031   Nope. Sleeping...
[02/11/2022 11:17:28] Query Log Analytics for: bar 1520676031   Got it in 2.02 minutes
#>
```



