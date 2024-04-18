---
layout: single
classes: wide
share: false
title: Get-ADGroupMember Forgot My Contacts
tags: [AD,PowerShell]
---

One of the first challenges I ran into with PowerShell Desired State Configuration (DSC) custom resources was the security context in which the custom resources were running (System).  The custom resources I was making needed to call FIM with a known security context.  This seemed fair to me, as I’d expect to do the same when calling other systems such as AD or SharePoint.
FIM requires a ‘Person’ object in the FIM Service for the person calling the web service endpoints.  Stated another way; the FIM Service in my demo environment did not know who the System account was, so it told DSC to go away with ‘access-denied’.
My workaround was to include a Credential parameter in all of my custom resources for DSC, then to use that inside my custom resources to make the FIM calls.  I did not like this at all, because it meant I had to have these credentials lying around and would then have to protect and update them (boo!).  This forced me to learn about how DSC protects credentials, which was quite useful but still not my favourite solution.
While demonstrating this at the MVP Summit, my co-worker and DS MVP (Brian Desmond) suggested that I just create the System account in the FIM Service.  I love working with smart people, too bad they get to suffer my foolishness!  Anyhow, I tried Brian’s suggestion and at first it didn’t work. 
My first attempt was to create the Person object in FIM using the computer name and its ObjectSID from AD.

```powershell
### Create a the Computer Account as a FIM Person
###
New-FimImportObject -State Create -ObjectType Person -Changes @{
    AccountName = (hostname)
    DisplayName = (hostname)
    Domain      = 'Redmond'
    ObjectSID   = (Get-ObjectSid -AccountName (hostname))
} -ApplyNow
```

This didn’t work, but looking at the error in the event log gave me a clue.  The error details were:
GetCurrentUserFromSecurityIdentifier: No such user NT AUTHORITY\SYSTEM, S-1-5-18
Looking at the ObjectSID and account details, it was not using the specific computer but instead the well-known name and ObjectSID.  The error suggested what FIM was looking for so I tried this:

```powershell
###
### Create a the NT AUTHORITY\SYSTEM as a FIM Person
###
New-FimImportObject -State Create -ObjectType Person -Changes @{
    AccountName = 'SYSTEM'
    DisplayName = 'SYSTEM'
    Domain      = 'NT AUTHORITY'
    ObjectSID   = (Get-ObjectSid -AccountName 'NT AUTHORITY\SYSTEM')
} -ApplyNow
```

It worked!  Now I can rip the credentials out of my custom DSC resources but I might leave them in and make them optional since I’ve already done most of the work.
