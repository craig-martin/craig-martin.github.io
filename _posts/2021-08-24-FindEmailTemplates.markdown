---
layout: single
classes: wide
share: false
title: Finding Email Templates in the MIM Service
tags: [PowerShell, MIM]
---
# Finding Email Templates
As a service engineer supporting a large MIM deployment I still get to do fun maintenance tasks even though there is not much feature work as we get closer to retiring the system.  

Today I needed to find all the Email Template objects with a specific string, and replace that string with a new string.  Sounds simple enough, but has a couple tricks. 

# XPath Filter Versus Where-Object Filter
The MIM Service enumeration endpoint accepts XPath queries, which are pretty powerful but have some limitations.  
One of the limitations is that the contains() function is not supported on large string attributes such as XOML or EmailBody.
For this task I need to search those large string attributes to:
* find the workflow definitions that contain email templates (by inspecting the XOML attribute)
* find the email templates that contain the string to be replaced (by inspecting the EmailBody attribute)

The workaround for that limitation is to get the objects from MIM using XPath, then to continue filtering them using the Where-Object cmdlet in PowerShell.

Here's what that looks like:

``` powershell
<#
Script to update email templates to replace https://foo/ with https://foo.bar.com/
NOTE: this must be run on a computer running the FimService
#>
# Array to store the Object IDs of Email Template objects found in Workflow Definitions
$emailTemplatesIDsInUse = @()
# Search Authorization workflows with Approval Activities
# Get the IDs of the Email Templates from the XOML
Search-Resources -XPath "/ManagementPolicyRule[Disabled=false]/AuthorizationWorkflowDefinition" -AttributesToGet DisplayName, XOML | ForEach-Object {
    $xoml = $PSItem.XOML -as [XML]
    if ($xoml.SequentialWorkflow.ApprovalActivity )
    {
        Write-Host "Workflow $($PSItem.DisplayName) Has ApprovalActivity "
        $emailTemplatesIDsInUse += $xoml.SequentialWorkflow.ApprovalActivity.ApprovalDeniedEmailTemplate
        $emailTemplatesIDsInUse += $xoml.SequentialWorkflow.ApprovalActivity.ApprovalTimeoutEmailTemplate
        $emailTemplatesIDsInUse += $xoml.SequentialWorkflow.ApprovalActivity.ApprovalCompleteEmailTemplate
        $emailTemplatesIDsInUse += $xoml.SequentialWorkflow.ApprovalActivity.ApprovalEmailTemplate
        $emailTemplatesIDsInUse += $xoml.SequentialWorkflow.ApprovalActivity.EscalationEmailTemplate        
    }
}
# Search Action workflows with Email Notification Activities
# Get the ID of the Email Templates from the XOML
Search-Resources -XPath "/ManagementPolicyRule[Disabled=false]/ActionWorkflowDefinition" -AttributesToGet DisplayName, XOML | ForEach-Object {
    $xoml = $PSItem.XOML -as [XML]
    if ($xoml.SequentialWorkflow.EmailNotificationActivity)
    {
        Write-Host "Workflow $($PSItem.DisplayName) Has EmailNotificationActivity"
        $emailTemplatesIDsInUse += $xoml.SequentialWorkflow.EmailNotificationActivity.EmailTemplate
    }
}
# From the IDs, get the Email Template objects
$emailTemplates = $emailTemplatesIDsInUse | select -Unique | ForEach-Object {
    Get-Resource -ID $PSItem        
}
# Create a backup file for each Email Template 
New-Item -Name EmailTemplateBackup -ItemType Directory
Set-Location -Path .\EmailTemplateBackup
$emailTemplates | ForEach-Object {
    $PSItem.EmailBody | Out-File -FilePath ($PSItem.ObjectID.Value + '.html')
}
# Replace https://foo/ with https://foo.bar.com/
# Save the Email Template to MIM
$emailTemplates | Where-Object EmailBody -Like '*http*' | ForEach-Object {
    $PSItem.EmailBody = $PSItem.EmailBody -replace 'https://foo/', 'https://foo.bar.com/'
    $PSItem | Save-Resource
}

```
