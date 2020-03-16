---
layout: single
classes: wide
title:  "Using LithnetRMA to Update MIM Group Membership"
date:   2020-01-31 19:20:42 -0800
share: false
tags: [MIM, Lithnet, PowerShell]
---

Today I needed to reproduce a Microsoft Identity Manager (MIM) workflow issue when users were added to groups.  My tolerance for doing it in the MIM Portal waned so I decided to try using the LithnetRMA PowerShell module to do it.  In the past I would always use the FIM PowerShell Module for such tasks but now LithnetRMA is an option so I wanted to learn more about it.

Things I like about the LithnetRMA module:

* **Open Source** - the code is available on GitHub, even has a solid wiki!
* **PowerShell Gallery** - the module can be installed from the PowerShell Gallery (Install-Module -Name LithnetRMA)
* **Code Quality** - pretty great looking code
* **Integrates with the MIM Web Services** - does not depend on the FIMAutomation PowerShell Snap-In, which makes it more reliable and fast

The experiment was a success!  I'm really happy with how easy it was to use.  Here's the sample:

```powershell
<#
Install the module - choosing CurrentUser Scope so it does not require Administrator privilege
#>
Install-Module -Name LithnetRMA -Scope CurrentUser

<#
Get the user and group objects
#>
$user  = Get-Resource -ObjectType Person -AttributeName AccountName -AttributeValue cmart
$group = Get-Resource -ObjectType Group  -AttributeName DisplayName -AttributeValue cmartppedg5000

<#
Add the user to the group
#>
$group.ExplicitMember.Add($user.ObjectID)
Save-Resource -Resources $group

<#
Remove the user from the group
#>
$group.ExplicitMember.Remove($user.ObjectID)
Save-Resource -Resources $group
```
