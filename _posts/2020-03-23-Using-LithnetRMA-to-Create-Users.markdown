---
layout: single
classes: wide
title:  "Using Lithnet RMA to Create a MIM Person"
share: false
tags: [MIM, Lithnet, PowerShell]
---

Most of the time MIM gets Person objects created by the Synchronization Service, but sometimes it is useful to create them directly in the MIM Service, for example in development environments.

The sample below shows how to use the [LithnetRMA](https://github.com/lithnet/resourcemanagement-powershell/wiki) to create a new Person object with enough attributes for it to access the MIM Service.  The only trick to this scenario is the ObjectSID attribute.  If that attribute is missing or incorrect then the MIM Service will deny access.

```powershell
<#
Install the module - choosing CurrentUser Scope so it does not require Administrator privilege
#>
Install-Module -Name LithnetRMA -Scope CurrentUser

<#
Get the security identifier of the domain user
    This is used by MIM to authorize the user
#>
$ntaccount = New-Object System.Security.Principal.NTAccount 'Litware\Amanda'
try
{
	$securityIdentifier = $ntaccount.Translate([System.Security.Principal.SecurityIdentifier]) 
}
catch
{    
	Throw "Account could not be resolved to a SecurityIdentifier"
}

<#
Get the Byte Array form of the SID
    The LithnetRMA property of type Binary expects a Btye array
#>
$securityIdentifierBytes = new-object System.Byte[] -argumentList $securityIdentifier.BinaryLength
$securityIdentifier.GetBinaryForm($securityIdentifierBytes, 0)

<#
Create the new Person object in memory, then save it to MIM
    Note: the $newPerson object enjoys some nice intellisense if type type '.' after the variable name
#>
$newPerson = New-Resource -ObjectType Person
$newPerson.AccountName  = 'Amanda'
$newPerson.Domain       = 'Litware'
$newPerson.DisplayName  = 'Amanda Hugnkiss'
$newPerson.ObjectSID    = $securityIdentifierBytes
Save-Resource $newPerson 

<#
Review the object by getting it back from the MIM Service
#>
Get-Resource -ObjectType Person -AttributeName AccountName -AttributeValue 'cmart' -AttributesToGet ObjectSID,Domain,AccountName
<#
AccountName                      : Amanda
Domain                           : Litware
ObjectSID                        : {1, 5, 0, 0...}
ObjectType                       : Person
#>
```

