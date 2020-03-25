---
layout: single
classes: wide
share: false
title: How to Disable Azure Disk Encryption
tags:
- PowerShell
- Azure
- Azure Disk Encryption
- Security
---

There are a couple scanarios where I need to take an Azure Disk from a VM in one subscription and copy it to another subscription.  When the disk is encrypted (as disks should be) this error prevents copying the disk resource to another subscription:

```json
{
  "code": "ResourceMoveProviderValidationFailed",
  "message": "Resource move validation failed. Please see details. Diagnostic information: timestamp '20200325T203515Z', subscription id '00000000-0000-0000-0000-000000000005', tracking id '00000000-0000-0000-0000-00000000000B', request correlation id '00000000-0000-0000-0000-00000000000C'.",
  "details": [
    {
      "code": "CrossSubscriptionMoveOfEncryptedDisk",
      "target": "Microsoft.Compute/disks",
      "message": "The move resources request contains resources like /subscriptions/00000000-0000-0000-0000-000000000005/resourceGroups/TheDiskResourceGroup/providers/Microsoft.Compute/disks/SomeEncryptedDiskName which are encrypted. Please check details for these resource ids."
      ]
    }
  ]
}
```

While I appreciated the safety lock, I intend to attach the disk to a VM that is able to decrypt it, so just need to move the disk.

Here is a look at the disk:

```powershell
Get-AzDisk -ResourceGroupName TheDiskResourceGroup -DiskName SomeEncryptedDiskName | 
Select-Object -ExpandProperty EncryptionSettingsCollection 
<# --BEFORE--

Enabled EncryptionSettings                                                     EncryptionSettingsVersion
------- ------------------                                                     -------------------------
   True {Microsoft.Azure.Management.Compute.Models.KeyVaultAndSecretReference} 1.1                      

#>
```

To let Azure know that the disk is not encrypted, just disable the setting.  

```powershell
## Disable the encryption setting for the disk
Update-AzDisk -ResourceGroupName TheDiskResourceGroup -DiskName SomeEncryptedDiskName -DiskUpdate (
    New-AzDiskUpdateConfig -EncryptionSettingsEnabled $false
)
```

Looking at the disk again we can see that encryption is no longer enabled.  Note that the disk is still encrypted, but the Azure resource properties no longer reflect that state.

```powershell
Get-AzDisk -ResourceGroupName TheDiskResourceGroup -DiskName SomeEncryptedDiskName | 
Select-Object -ExpandProperty EncryptionSettingsCollection 
<# --AFTER--

Enabled EncryptionSettings EncryptionSettingsVersion
------- ------------------ -------------------------
  False {}                 1.1                      
                
#>
```
