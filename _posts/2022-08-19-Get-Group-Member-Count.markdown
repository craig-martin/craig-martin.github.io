---
layout: single
classes: wide
share: false
title: Get Group Member Count Using Graph and PowerShell
tags: [PowerShell,Graph]
---

Group count can be a useful piece of data, I use it sometimes to check the consistency of a group between on-premises and Azure Active Directory.

This sample issues two graph queries:
1. Get the group object using a filter
2. Get the group members and the member count

A bonus in this sample is the use of a VM to get the Azure AD access token.  I like this because:
- it requires no user interaction
- it requires no secrets, passwords or certificates

For more information about getting a token using the VM identity:
[Azure Instance Metadata Service](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service?tabs=windows)

``` powershell
# Get an access token using the VM identity
$AzureInstanceMetadataServiceUrl = 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01' 
try{
    $token = Invoke-RestMethod -Uri "$AzureInstanceMetadataServiceUrl&resource=https://graph.microsoft.com" -Headers @{Metadata=$true} -Verbose:$false
}
catch{
    throw "Failed to get AAD access token : $_"
}

# Get the group using a filter
$groupUrl = "https://graph.microsoft.com/v1.0/groups?`$filter=mailnickname eq 'PedalPushers'"
$groupResult = Invoke-RestMethod -Headers @{Authorization = "Bearer $($token.access_token)"} -Uri $groupUrl -Method Get

# Get the group member count using the group ID
$groupMemberCountUrl = "https://graph.microsoft.com/v1.0/groups/$($groupResult.value.id)/members?`$count=true&`$top=1"
$groupMemberCountResult = Invoke-RestMethod -Headers @{Authorization = "Bearer $($token.access_token)";ConsistencyLevel = 'Eventual'} -Uri $groupMemberCountUrl -Method Get
$groupMemberCountResult.'@odata.count'  
```
