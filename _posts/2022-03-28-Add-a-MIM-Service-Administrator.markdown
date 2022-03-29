---
layout: single
classes: wide
share: false
title: Add a MIM Service Administrator
tags: [MIM,PowerShell,Lithnet]
---

Short sample to show how to use PowerShell to add a member to the MIM Service Administrators Set.  

To run the script you must already be an administrator.

For least privilege it is handy to keep the number of MIM administrators to a minimum, even better to not hold the permission persistently.

``` powershell
# Get the Person object
$user = Get-Resource -ObjectType Person -AttributeName AccountName -AttributeValue <your account name>

# Get the Administrators Set
$set = Get-Resource -ObjectType Set -AttributeName DisplayName -AttributeValue Administrators

# Add a Member
$set.ExplicitMember.Add($user)

# Submit the Request
Save-Resource -Resources $set
```
