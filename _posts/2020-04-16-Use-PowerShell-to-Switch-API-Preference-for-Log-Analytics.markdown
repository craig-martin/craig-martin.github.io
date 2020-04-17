---
layout: single
classes: wide
title:  "Use PowerShell to Switch API preference for Log Alerts"
share: false
tags: [PowerShell, Azure, Log Analytics]
---

This post has script snippets to follow the steps in [Switch API preference for Log Alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-log-api-switch#process-of-switching-from-legacy-log-alerts-api).

The snippets just use [Invoke-RestMethod](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-restmethod) to call the URLs specified.  Each snippet uses a token obtained using this snippet:

``` powershell
<#
Credit for this snippet goes to:
https://www.thelazyadministrator.com/2019/07/22/connect-and-navigate-the-microsoft-graph-api-with-powershell/
#>
$clientId           = "1950a258-227b-4e31-a9cf-717495945fc2"  # This is the standard client ID for Windows Azure PowerShell
$redirectUrl        = [System.Uri]"urn:ietf:wg:oauth:2.0:oob" # This is the standard Redirect URI for Windows Azure PowerShell
$tenant             = "LITWARE.onmicrosoft.com"               # TODO - your tenant name goes here
$resource           = "https://management.core.windows.net"   
$authUrl            = "https://login.microsoftonline.com/$tenant";
$postParams         = @{ resource = "$resource"; client_id = "$clientId" }

$response = Invoke-RestMethod -Method POST -Uri "$authurl/oauth2/devicecode" -Body $postParams
Write-Host "Response: $($response.message)"

#I got tired of manually copying the code, so I did string manipulation and stored the code in a variable and added to the clipboard automatically
$code = ($response.message -split "code " | Select-Object -Last 1) -split " to authenticate."
Set-Clipboard -Value $code
Start-Process "https://aka.ms/devicelogin" # must complete before the rest of the snippet will work

# Get the initial token
$tokenParams = @{ 
    grant_type  = "device_code"
    resource    = $resource
    client_id   = $clientId
    code        = $response.device_code 
}

$tokenResponse = Invoke-RestMethod -Method POST -Uri "$authurl/oauth2/token" -Body $tokenParams
```

With authentication out of the way we can now call the APIs.

First, check the API version of your Log Analytics workspace.  Note the items that need to be put into the URL:
* Subscription ID - the GUID of your subscription
* Resource Group Name - the name of the resource group containing the Log Analytics workspace
* Log Analytics Workspace Name - the name of the workspace you want to check

``` powershell
$apiUrl = "https://management.azure.com/subscriptions/<YOUR SBUSCRIPTION GUID>/resourceGroups/<YOUR RESOURCE GROUP NAME>/providers/Microsoft.OperationalInsights/workspaces/<YOUR LOG ANALYTICS WORKSPACE NAME>/alertsversion?api-version=2017-04-26-preview"
Invoke-RestMethod -Headers @{Authorization = "Bearer $($tokenResponse.Access_Token)"} -Uri $apiUrl -Method Get 

<# Output should look like this:

version scheduledQueryRulesEnabled
------- --------------------------
      2                      False

#>
```

To switch the API preference the URL looks the same but this time we do a _Post_ and provide a _Body_ as specified by the article.

``` powershell
### Change the AlertsVersion
$apiUrl = "https://management.azure.com//subscriptions/<YOUR SBUSCRIPTION GUID>/resourceGroups/<YOUR RESOURCE GROUP NAME>/providers/Microsoft.OperationalInsights/workspaces/<YOUR LOG ANALYTICS WORKSPACE NAME>/alertsversion?api-version=2017-04-26-preview"
Invoke-RestMethod -Headers @{Authorization = "Bearer $($tokenResponse.Access_Token)"} -Uri $apiUrl -Method Put -Body '{"scheduledQueryRulesEnabled": true}' -UseBasicParsing -ContentType 'application/json'
<# OUTPUT should look like this:

version scheduledQueryRulesEnabled
------- --------------------------
      2                       True

#>
```

That's all there is to it.  Remember to avoid doing this until you've reviewed this GitHub issue:
[Changing API Preference results in Alert Rule names being overwritten with GUIDs #44936](https://github.com/MicrosoftDocs/azure-docs/issues/44936)