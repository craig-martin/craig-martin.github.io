---
layout: single
classes: wide
title:  "Constructing the DN for the Azure AD Connector"
date:   2020-03-10 19:20:42 -0700
categories: blog update
share: false
tags: [Azure AAD Connect, Synchronization, PowerShell]
---

Searching a for a Connector Space Object in an Active Directory connector is pretty simple because you can search by:
* Distinguished Name
* Relative Distinguished Name
* Sub Tree

Those criteria are pretty easy to find because they are attributes on the AD objects exposed using a wide assortment of tools, just to name a few:
* Active Directory Users and Computers
* Ldp.exe
* PowerShell ActiveDirectory Module

Searching for a CSObject in the Azure Active Directory connector space is difficult because the DN is not simply an attribute from the AAD object, it is constructed and looks something like this:

```
CN={714E52566836554B77555772666F674F3675363846413D3D}
```

*Update - 2020-03-18*


The DN above is constructed using the sourceAnchor attribute.  The sourceAnchor attribute is constructed in at least two ways:
1. If the object is cloud-mastered (DeviceTrustType = AzureAD) then sourceAnchor recipe is:
```powershell
'Device_<AAD ObjectId attribute>'
```

2. If the object is mastered on-premises (DeviceTrustType = ServerAD), then
```powershell
[System.Convert]::ToBase64String(([GUID]'AAD DeviceId attribute').ToByteArray())
```

from the anchor attribute, in my case it is a Device object and the anchor attribute is the DeviceId attribute:

``` powershell
Get-AzureADDevice -SearchString icemelted | Select-Object *
<#
ObjectId                      : c4da741b-5eff-4cdb-adc9-b66d22e3cb9d
ObjectType                    : Device
AccountEnabled                : True
DeviceId                      : 8755d4a8-0aa5-45c1-ab7e-880eeaeebc14
DeviceOSType                  : Windows
DeviceOSVersion               : 10.0.18363.720
DeviceTrustType               : ServerAd
DisplayName                   : icemelted
#>
```

With the DeviceId attribute I can now construct the DN then search for the object in the AAD Connector Space.  If there were only a handful of objects this would not be necessary but as the number of objects grows it gets really difficult to find objects without knowing the DN.

``` powershell
# Handy little module
Import-Module 'C:\Program Files\Microsoft Azure AD Sync\Extensions\AADConnector.psm1'

# the Device ID of the AAD Device object
$anchor = [GUID]'8755d4a8-0aa5-45c1-ab7e-880eeaeebc14' 
$anchorBase64String = [System.Convert]::ToBase64String($anchor.ToByteArray())

# Get the DN from the Base64 encoded anchor attribute value
$DN = ConvertTo-ADSyncAadDistinguishedName -sourceAnchor $anchorBase64String

# Try getting the AAD CSObject by the constructed DN
Get-ADSyncCSObject -ConnectorName 'hoofhearted.onmicrosoft.com - AAD' -DistinguishedName $DN

```

The last line in the sample above uses the Get-ADSyncCSObject to search for the object but you could also use the Synchronization Service Manager.  Happy searching!