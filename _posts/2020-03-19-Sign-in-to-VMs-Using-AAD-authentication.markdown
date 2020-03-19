---
layout: single
classes: wide
title:  "Sign in to Windows VMs in Azure using Azure AD authentication"
share: false
tags: [Azure, Azure Active Directory, Active Directory, Security]
---

Joining servers to domains is something I've just done for decades.  It's obvious everything is moving to the cloud but I'm tickled by this:

[Sign in to Windows virtual machine in Azure using Azure Active Directory authentication (Preview)](https://docs.microsoft.com/en-us/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows)

To domain join a computer to Active Directory you typically need to provide AD credentials and have connectivity do a domain controller.

To join a computer to Azure Active Directory you just need to enable the Azure AD login option on the Azure Virtual Machine, then add two RBAC role assignments.

Once the computer joins successfully it is visible in the Azure Active Directory blade from the Azure Portal, or using PowerShell:

```powershell
Get-AzureADDevice -SearchString newServer2019vm | Select-Object *
<#
ObjectId                      : <oops I guid it again>
ObjectType                    : Device
AccountEnabled                : True
AlternativeSecurityIds        : {class AlternativeSecurityId {
                                  IdentityProvider: 
                                  Key: System.Byte[]
                                  Type: 2
                                }
                                }
ApproximateLastLogonTimeStamp : 3/19/2020 6:14:02 PM
ComplianceExpiryTime          : 
DeviceId                      : <oops I guid it again>
DeviceMetadata                : 
DeviceObjectVersion           : 2
DeviceOSType                  : Windows
DeviceOSVersion               : 10.0.17763.0
DevicePhysicalIds             : {[AzureResourceId]:/subscriptions/<oops I guid it again>/resourceGroups/vmResourceGroup/providers/Microsoft.Compute/virtualMachines/newServer2019vm, 
                                [USER-GID]:foo:bar, [GID]:g:bar, [USER-HWID]:foo:bar...}
DeviceTrustType               : AzureAd
DirSyncEnabled                : 
DisplayName                   : newServer2019vm
IsCompliant                   : 
IsManaged                     : 
LastDirSyncTime               : 
ProfileType                   : SecureVM
SystemLabels                  : {}
#>
```

Here is the log output, showing the join process:

```
Execution Output:
[3/19/2020 6:13:49 PM] 10.0.17763.1 (WinBuild.160101.0800)
[3/19/2020 6:13:49 PM] Adding Registry Settings.
[3/19/2020 6:13:49 PM] Enabling PKU2U policy.
[3/19/2020 6:13:59 PM] Starting AAD Join process.
[3/19/2020 6:14:05 PM] dsregcmd::wmain logging initialized.
[3/19/2020 6:14:05 PM] Check existing join status.
[3/19/2020 6:14:05 PM] Join Azure with MSI credential.
[3/19/2020 6:14:05 PM] Getting Azure VM metadata.
[3/19/2020 6:14:05 PM] Targeting host name:169.254.169.254, url path: /metadata/instance/compute?api-version=2017-12-01
[3/19/2020 6:14:05 PM] DsrCmdAzureHelper::GetMetadataRestResponse: HTTP Status Code: 200
[3/19/2020 6:14:05 PM] dwDownloaded:463, dwCombinedSize:463
[3/19/2020 6:14:05 PM] dwDownloaded:0, dwCombinedSize:463
[3/19/2020 6:14:05 PM] Received Content (size 463):
[3/19/2020 6:14:05 PM] {"location":"westus2","name":"cmartDev031900","offer":"WindowsServer","osType":" ... d_D2s_v3","zone":""}
[3/19/2020 6:14:05 PM] Azure resource Id:/subscriptions/<guid>/resourceGroups/vmDeployTest/providers/Microsoft.Compute/virtualMachines/cmartDev031900
[3/19/2020 6:14:05 PM] Getting Tenant ID from MSI.
[3/19/2020 6:14:05 PM] Targeting host name:169.254.169.254, url path: /metadata/identity/info?api-version=2018-02-01
[3/19/2020 6:14:05 PM] DsrCmdAzureHelper::GetMetadataRestResponse: HTTP Status Code: 200
[3/19/2020 6:14:05 PM] dwDownloaded:51, dwCombinedSize:51
[3/19/2020 6:14:05 PM] dwDownloaded:0, dwCombinedSize:51
[3/19/2020 6:14:05 PM] Received Content (size 51):
[3/19/2020 6:14:05 PM] {"tenantId":"<oops I guid it again>"}
[3/19/2020 6:14:05 PM] Discover tenant info with Tenant ID <oops I guid it again>.
[3/19/2020 6:14:05 PM] Getting MSI token for app urn:ms-drs:enterpriseregistration.windows.net.
[3/19/2020 6:14:05 PM] Targeting host name:169.254.169.254, url path: /metadata/identity/oauth2/token?resource=urn:ms-drs:enterpriseregistration.windows.net&api-version=2018-02-01
[3/19/2020 6:14:05 PM] DsrCmdAzureHelper::GetMetadataRestResponse: HTTP Status Code: 200
[3/19/2020 6:14:05 PM] dwDownloaded:1487, dwCombinedSize:1487
[3/19/2020 6:14:05 PM] dwDownloaded:0, dwCombinedSize:1487
[3/19/2020 6:14:05 PM] Received Content (size 1487):
[3/19/2020 6:14:05 PM] {"access_token":".. ... oken_type":"Bearer"}
[3/19/2020 6:14:05 PM] Start join process with MSI credential.
[3/19/2020 6:14:05 PM] Join request ID: <oops I guid it again>
[3/19/2020 6:14:05 PM] Join response time: Thu, 19 Mar 2020 18:14:03 GMT
[3/19/2020 6:14:05 PM] Join HTTP status: 200
[3/19/2020 6:14:05 PM] Successfully joined machine to AAD.

```

Once the computer was AAD joined I was able to use RDP to login using my smart card (it also worked using Hello For Business).  Cool!

Gotta say I really like this experience compared to AD domain joining.  Not sure I'll be using it just yet in production because the management experience is quite different from what gets applied to computers today by SCCM and Group Policy.