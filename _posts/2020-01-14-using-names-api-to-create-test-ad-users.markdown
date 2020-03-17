---
layout: single
classes: wide
share: false
title: Using a Names API to Create Test AD Users
tags:
- Active Directory
- PowerShell
---

Found this cool little [API that provides names](http://names.drycodes.com/), made for a nice and quick little script to throw a bunch of test users into a test Active Directory domain.

```powershell
### Create a SecureString for the AccountPassword
$newPassword = ConvertTo-SecureString 'N0BoatBrewing!' -AsPlainText -Force

### Get 100 names
$namesContent = Invoke-RestMethod -Uri 'http://names.drycodes.com/100?separator=space&nameOptions=boy_names'

$namesContent.GetEnumerator() | ForEach-Object {
    ### Split the "FirstName LastName" string into two variables
    $FirstName, $LastName = $PSItem -split ' '   

    ### Construct a couple of strings
    $AccountName = "$FirstName $LastName"
    $DisplayName = "$($FirstName) $($LastName)"

    ### Create the AD User
    New-ADUser -Name $AccountName -GivenName $FirstName -Surname $LastName -SamAccountName $AccountName -DisplayName $DisplayName -Enabled $true -AccountPassword $newPassword -Path 'OU=People,DC=litware,DC=ca'

}
```