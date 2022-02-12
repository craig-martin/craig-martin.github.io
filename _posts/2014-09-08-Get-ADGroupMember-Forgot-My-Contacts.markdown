---
layout: single
classes: wide
share: false
title: Get-ADGroupMember Forgot My Contacts
tags: [AD,PowerShell]
---

Working on a script to copy groups and members from one forest to another, and was so happy with the Get-ADGroupMember cmdlet but ran into an issue that means I can’t use it.

The challenge was that the cmdlet wasn’t returning the correct group membership.  I could count the group members using this:

 
```powershell
Get-ADGroup hoofhearted -Properties member |
Select-Object -ExpandProperty member
```
 

However, this command would return no objects:

```powershell
Get-ADGroup hoofhearted |
Get-ADGroupMember
```
 

Thought it just a permissions issue, that I had access to the group but not the member objects, but that turned out to be false.

The answer is right in the [TechNet documentation for the Get-ADGroupMember cmdlet](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee617193(v=technet.10)):

> The Get-ADGroupMember cmdlet gets the members of an Active Directory group. Members can be users, groups, and computers.

Note that members can be users, groups and computers (NOT contacts).  Bummer, I guess they designed the cmdlet with security principals in mind so didn’t include contacts.

The workaround isn’t very difficult:

 
```powershell
Get-ADGroup hoofhearted -Properties member |
Select-Object -ExpandProperty member |
Get-ADObject
```
 

This is one of the things I love about PowerShell, and one of the reasons they don’t take many bugs, because there is almost always an easy workaround. 
