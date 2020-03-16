---
layout: single
classes: wide
title:  "Who Can Reset My Active Directory Password?"
share: false
tags: [Active Directory, PowerShell, Security]
---

Needed to figure out who was able to reset a specific Active Directory user password and turned to ldp.exe but wanted to try using PowerShell to reduce the clicks for next time.  Came up with this:

```powershell
<#
Who can reset my AD password?

Using the ActiveDirectory PowerShell module to get the nTSecurityDescriptor
Find the Access using the extended right for password reset
(see https://docs.microsoft.com/en-us/windows/win32/adschema/r-user-force-change-password)
#>
$user = Get-ADUser -Identity myUser -Properties nTSecurityDescriptor

$user.ntSecurityDescriptor.Access |
Where-Object ActiveDirectoryRights -Contains ExtendedRight |
Where-Object ObjectType -EQ '{00299570-246d-11d0-a768-00aa006e0529}' |
Select-Object AccessControlType, IdentityReference

<#
AccessControlType IdentityReference                
----------------- -----------------                
             Deny LITWARE\DENY-PasswordReset            
            Allow LITWARE\SuperTrustedPeople
            Allow LITWARE\NothingToSeeHereFolks                 
#>
```

In ldp.exe the extended rights are listed quite nicely, such as 'password reset'.  In the ntSecurityDescriptor property we get a nice object with useful properties but the extended access rights require you to know the ID of the access right, and the fact that it is stored in the ObjectType property.  Once that is figured out, a Where-Object filter can be used to find who has that right.
