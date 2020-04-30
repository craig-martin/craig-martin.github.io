---
layout: single
classes: wide
title:  "View SPF Details from DNS Using PowerShell"
share: false
tags: [PowerShell, DNS, SPF]
---

[SPF records](https://en.wikipedia.org/wiki/Sender_Policy_Framework) are something I am working with this week and needed to look at some so I figured PowerShell would be a fun way to do it.  Here's the short snippet:

``` powershell
Resolve-DnsName -Name pinkbike.com -Type TXT | 
Where-Object Strings -like v=* | 
Select-Object -ExpandProperty Strings

<#
v=spf1
ip4:198.90.5.0/24
include:_spf.google.com
include:mailgun.org 
-all
v=spf1
include:servers.mcsv.net ?all
#>
```