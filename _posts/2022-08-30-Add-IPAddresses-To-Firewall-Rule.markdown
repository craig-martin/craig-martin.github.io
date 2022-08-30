---
layout: single
classes: wide
share: false
title: Add IP Addresses to a Firewall Rule
tags: [PowerShell]
---

Some of our services are locked down with a list of approved IP addresses.  It can be a pain to manage sometimes using the firewall UX so this snippet comes in handy.

For this snippet I use an extra array because I was unable to get $FirewallAddressFilter.RemoteAddress.Add('1.2.3.4') to work, other than that it works great. 

It also provides a nice backup of the IP addresses before the change is made.

``` powershell

<# Get a list of firewall rules, helpful to find the rule to modify
Get-NetFirewallRule | Where Enabled -eq $true  | select Name, DisplayName, Enabled | sort DisplayName
#>

# Get the Firewall rule
$rule = Get-NetFirewallRule -DisplayName 'My Firewall Rule' 

# Get the Address Filter for the Firewall Rule
$FirewallAddressFilter = $rule | Get-NetFirewallAddressFilter

# Export the existing address filter to a file, just in case
$FirewallAddressFilter | ConvertTo-Json | Out-File (Join-Path $home FirewallAddressFilter.json)

# Show the current number of addresses
$FirewallAddressFilter.RemoteAddress.Count

# Create a new array using the existing addresses
$newRemoteAddressArray = $FirewallAddressFilter.RemoteAddress

# Add addresses to the array
$newRemoteAddressArray += @(
'1.2.3.4'
'1.2.3.5'
'1.2.3.6'
)

# Show the future number of addresses
$newRemoteAddressArray.Count

# Replace the current addresses with the future addresses
$FirewallAddressFilter.RemoteAddress = $newRemoteAddressArray

# Save the Address Filter
$FirewallAddressFilter | Set-NetFirewallAddressFilter 

```
