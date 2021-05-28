---
layout: single
classes: wide
share: false
title: Converting Log Analytics Query Responses to PowerShell PSCustomObjects
tags: [PowerShell, Azure, Log Analytics]
---
# Log Analytics Query API
Getting to be a fan of the [Log Analytics Query API](https://dev.loganalytics.io/documentation/Using-the-API) because it enables queries over just HTTP without other dependencies.  The response format can be a little weird so I created a little function to convert the response rows to a PowerShell custom object.  The response format seems to optimize by specifying column names separately from row values.  Makes sense, because the resulting JSON response is likely a ton smaller but it comes at a small usability cost, hence this function.

# ConvertLogAnalyticsQueryRespose-ToPSObject Function
The function loops through three collections:
* Tables - there is normally just one table
* Columns - the list of column names
* Rows - the array of rows, each row containts an array of values matching the index of Columns

Each row is output as a custom object.

``` powershell
function ConvertLogAnalyticsQueryRespose-ToPSObject
{
<#
.Synopsis
   Converts Log Analytics query API responses to PowerShell custom objects
.DESCRIPTION
   https://dev.loganalytics.io/documentation/Using-the-API
.EXAMPLE
    $logQuery = "Event | take 100"
    $logQueryBody = @{"query" = $logQuery} | ConvertTo-Json
    $logQueryHeaders = @{
        Authorization  = "Bearer $($token.access_token)"
        "Content-Type" = "application/json"
    }
    $logQueryUri = "https://westus2.api.loganalytics.io/v1/workspaces/$($logAnalyticsWorkspaceID)/query"

    $queryResponse = Invoke-RestMethod -Method POST -Uri $logQueryUri -Headers $logQueryHeaders -Body $logQueryBody 

    $queryResponse | ConvertLogAnalyticsQueryRespose-ToPSObject
#>
    [CmdletBinding()]
    Param
    (
        # Tables property of the Log Analytics query reponse
        [Parameter(Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
        $Tables
    )

    for ($t = 0; $t -lt $Tables.Count; $t++)
    { 
        for ($r = 0; $r -lt $Tables[$t].rows.Count; $r++)
        {
            $psCustomObjectHashTable = @{}
            for ($c = 0; $c -lt $Tables[$t].columns.Count; $c++)
            { 
                $psCustomObjectHashTable.Add($Tables[$t].columns[$c].Name, $Tables[$t].rows[$r][$c])
            }
            [pscustomobject]$psCustomObjectHashTable | Write-Output
        } 
    }
}
```
