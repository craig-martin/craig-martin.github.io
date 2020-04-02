---
layout: single
classes: wide
title:  "Using PowerShell DSC to Set SharePoint ViewState"
share: false
tags: [MIM, PowerShell, SharePoint, DSC]
---

Been working on using DSC to install MIM and been wanting to use as much of the [SharePointDsc PowerShell DSC resource module](https://github.com/dsccommunity/SharePointDsc) as possible.  

For referece the steps to automate are listed here: [Set up an identity management server: SharePoint](https://docs.microsoft.com/en-us/microsoft-identity-manager/prepare-server-sharepoint).

I was able to get all the SharePoint steps done using the SharePointDsc module except one, which I capture here.

The nagging item was to disable the SharePoint Server-Side Viewstate.  If this is *not* done then the MIM Service and Portal installation will fail because it checks this setting.

To set the ViewState I just use the SharePoint PowerShell Snap-In in a Script module.  In general I avoid the script module but this case was so simple.

``` powershell
configuration TestSPContentServiceViewState
{
    Import-DscResource –ModuleName PSDesiredStateConfiguration

    node (hostname)
    {
        Script SPContentServiceViewState
        {
            #DependsOn = '[SPSite]MimSite'

            GetScript = {Return "SPContentServiceViewState"}
            TestScript = {  
                add-pssnapin Microsoft.SharePoint.PowerShell
                $contentService = [Microsoft.SharePoint.Administration.SPWebService]::ContentService

                if ($contentService.ViewStateOnServer -eq $false)
                {
                    Write-Verbose "ViewStateOnServer is false (as desired), returning True"
                    return $true
                }
                else
                {
                    Write-Verbose "ViewStateOnServer is true (undesirable), returning False"
                    return $false
                }
            }
            SetScript = {
                add-pssnapin Microsoft.SharePoint.PowerShell

                Write-Verbose "Configuring View State..."
                $contentService = [Microsoft.SharePoint.Administration.SPWebService]::ContentService
                $contentService.ViewStateOnServer = $false
                $contentService.Update()
            }
        }
    }
}

TestSPContentServiceViewState -OutputPath C:\Setup\TestSPContentServiceViewState 

Start-DscConfiguration -Verbose -Wait -Path C:\Setup\TestSPContentServiceViewState -Force
```

Here is the log output:

``` text
VERBOSE: Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = SendConfigurationApply,'className' = MSFT_DSCLocalConfigurationManager,'namespaceName' = root/Microsoft/Windows/DesiredStateConfiguration'
.
VERBOSE: An LCM method call arrived from computer HoofHearted with user sid S-007.
VERBOSE: [HoofHearted]: LCM:  [ Start  Set      ]
VERBOSE: [HoofHearted]: LCM:  [ Start  Resource ]  [[Script]SPContentServiceViewState]
VERBOSE: [HoofHearted]: LCM:  [ Start  Test     ]  [[Script]SPContentServiceViewState]
VERBOSE: [HoofHearted]:                            [[Script]SPContentServiceViewState] ViewStateOnServer is true (undesirable), returning False
VERBOSE: [HoofHearted]: LCM:  [ End    Test     ]  [[Script]SPContentServiceViewState]  in 1.1880 seconds.
VERBOSE: [HoofHearted]: LCM:  [ Start  Set      ]  [[Script]SPContentServiceViewState]
VERBOSE: [HoofHearted]:                            [[Script]SPContentServiceViewState] Performing the operation "Set-TargetResource" on target "Executing the SetScript with the user supplied credential".
VERBOSE: [HoofHearted]:                            [[Script]SPContentServiceViewState] Configuring View State...
VERBOSE: [HoofHearted]: LCM:  [ End    Set      ]  [[Script]SPContentServiceViewState]  in 0.1720 seconds.
VERBOSE: [HoofHearted]: LCM:  [ End    Resource ]  [[Script]SPContentServiceViewState]
VERBOSE: [HoofHearted]: LCM:  [ End    Set      ]
VERBOSE: [HoofHearted]: LCM:  [ End    Set      ]    in  2.6100 seconds.
VERBOSE: Operation 'Invoke CimMethod' complete.
VERBOSE: Time taken for configuration job to complete is 2.736 seconds

```

In the log output above the Test function in DSC found the ViewState to be in the wrong state, so it executed the Set function to put the item into the desired state.