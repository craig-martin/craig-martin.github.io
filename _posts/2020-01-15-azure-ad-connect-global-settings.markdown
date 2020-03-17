---
layout: single
classes: wide
share: false
title: Azure AD Connect Global Settings
tags:
- PowerShell
- ADSync
- Azure
- AAD
---

Sync deployments always have some configuration settings hanging around, and usually end up in XML files somewhere on the computer running the synchronization service because there has not been a good way to store configuration settings in the synchronization service itself.  That seems to have changed in Azure AD Connect with the Global Settings.  Global settings are not well documented but they appear to work pretty well.  From what I've observed they are saved to the ADSync database and survive computer restarts.  Here's a tour...  

The command for getting the global settings is Get-ADSyncGlobalSettings.   

```powershell
Get-Help Get-ADSyncGlobalSettings -Full
<#
NAME
    Get-ADSyncGlobalSettings
    
SYNTAX
    Get-ADSyncGlobalSettings [-WhatIf] [-Confirm]  []
    
    
PARAMETERS
    -Confirm
        
        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false
        
    -WhatIf
        
        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false
        
    
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see 
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).   
INPUTS
    None
    
OUTPUTS
    System.Object
    
ALIASES
    None
    
REMARKS
    None
#>
```

The help content is minimalist but like any good command it can be run without input.

```powershell
Get-ADSyncGlobalSettings
<#
Version          : 396
SqlSchemaVersion : 615
InstanceId       : d112f5f4-4248-4959-8dc8-eac88e531433
Schema           : Microsoft.IdentityManagement.PowerShell.ObjectModel.Schema
Parameters       : {Microsoft.Synchronize.SynchronizationPolicy, Microsoft.SynchronizationOption.JoinCriteria, Microsoft.UserSignIn.DesktopSsoEnabled, Microsoft.Synchronize.MaintenanceEnabled...}
#>
```

OK, looks like some version detail and a property named Parameters containing the actual settings.  Get-Member sometimes reveals other useful properties and methods, and also tells us the type name:

```powershell
Get-ADSyncGlobalSettings | Get-Member
<#
   TypeName: Microsoft.IdentityManagement.PowerShell.ObjectModel.GlobalSettings

Name                               MemberType Definition                                                                                                                                                                                                         
----                               ---------- ----------                                                                                                                                                                                                         
AddOrReplaceConfigurationParameter Method     void AddOrReplaceConfigurationParameter(string parameterName, string parameterValue), void AddOrReplaceConfigurationParameter(Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter parameter)
Equals                             Method     bool Equals(System.Object obj)                                                                                                                                                                                     
GetHashCode                        Method     int GetHashCode()                                                                                                                                                                                                  
GetSchema                          Method     System.Xml.Schema.XmlSchema GetSchema(), System.Xml.Schema.XmlSchema IXmlSerializable.GetSchema()                                                                                                                  
GetType                            Method     type GetType()                                                                                                                                                                                                     
IsStagingModeEnabled               Method     bool IsStagingModeEnabled()                                                                                                                                                                                        
ReadXml                            Method     void ReadXml(System.Xml.XmlReader reader), void IXmlSerializable.ReadXml(System.Xml.XmlReader reader)                                                                                                              
ToString                           Method     string ToString()                                                                                                                                                                                                  
WriteXml                           Method     void WriteXml(System.Xml.XmlWriter writer), void WriteXml(System.Xml.XmlWriter writer, bool legacyFormat), void IXmlSerializable.WriteXml(System.Xml.XmlWriter writer)                                             
InstanceId                         Property   guid InstanceId {get;set;}                                                                                                                                                                                         
Parameters                         Property   Microsoft.IdentityManagement.PowerShell.ObjectModel.ParameterKeyedCollection Parameters {get;set;}                                                                                                                 
Schema                             Property   Microsoft.IdentityManagement.PowerShell.ObjectModel.Schema Schema {get;set;}                                                                                                                                       
SqlSchemaVersion                   Property   int SqlSchemaVersion {get;set;}                                                                                                                                                                                    
Version                            Property   int Version {get;set;}                     
#>
```

Hmm, AddOrReplaceConfigurationParameter looks useful, more on that later in this post..

Looking now at the Parameters property...

```powershell
Get-ADSyncGlobalSettings | Select-Object -ExpandProperty Parameters
<#
Name                   : Microsoft.OptionalFeature.UserWriteBack
InputType              : String
Scope                  : SynchronizationGlobal
Description            : 
RegexValidationPattern : 
DefaultValue           : 
Value                  : False
Extensible             : False
PageNumber             : 0
Intrinsic              : False
DataType               : String

Name                   : Microsoft.Synchronize.StagingMode
InputType              : String
Scope                  : SynchronizationGlobal
Description            : 
RegexValidationPattern : 
DefaultValue           : 
Value                  : False
Extensible             : False
PageNumber             : 0
Intrinsic              : False
DataType               : String
#>
```

Each Parameter seems to have a lot of properties, but when viewed with Out-GridView it seems only the Name and Value properties change, the other properties are the same for all Parameters.  Spoiler alert: ADSync does let you specify DataType other than String but it seems to be hard coded to only support String.

Here are the names and values:

```powershell
Get-ADSyncGlobalSettings | 
Select-Object -ExpandProperty Parameters | 
Format-Table -AutoSize -Property Name, Value, DataType
<#
Name                                                   Value                         DataType
----                                                   -----                         --------
Microsoft.Synchronize.SynchronizationPolicy            Delta                           String
Microsoft.SynchronizationOption.JoinCriteria           AlwaysProvision                 String
Microsoft.UserSignIn.DesktopSsoEnabled                 False                           String
Microsoft.Synchronize.MaintenanceEnabled               True                            String
Microsoft.OptionalFeature.ExportDeletionThresholdValue 10                              String
Microsoft.Version.SynchronizationRuleImmutableTag      V1                              String
Microsoft.SynchronizationOption.AnchorAttribute        mS-DS-ConsistencyGuid           String
Microsoft.OptionalFeature.DirectoryExtensionAttributes                                 String
Microsoft.OptionalFeature.FilterAAD                    False                           String
Microsoft.GroupWriteBack.Forest                                                        String
Microsoft.GroupWriteBack.Container                                                     String
Microsoft.SynchronizationOption.UPNAttribute           userPrincipalName               String
Microsoft.Synchronize.SchedulerSuspended               False                           String
Microsoft.OptionalFeature.DirectoryExtension           False                           String
Microsoft.SynchronizationOption.CustomAttribute                                        String
Microsoft.Synchronize.TimeInterval                     00:30:00                        String
Microsoft.Synchronize.ServerConfigurationVersion       1.4.18.0                        String
Microsoft.SystemInformation.MachineRole                RolePrimaryDomainController     String
Microsoft.AADFilter.AttributeExclusionList                                             String
Microsoft.OptionalFeature.DeviceWriteBack              False                           String
Microsoft.OptionalFeature.AutoUpgradeState             Enabled                         String
Microsoft.Synchronize.NextStartTime                    Wed, 15 Jan 2020 21:57:01 GMT   String
Microsoft.Synchronize.RunHistoryPurgeInterval          7.00:00:00                      String
Microsoft.OptionalFeature.GroupFiltering               False                           String
Microsoft.ConnectDirectories.WizardDirectoryMode       AD                              String
Microsoft.Synchronize.SynchronizationSchedule          False                           String
Microsoft.OptionalFeature.ExchangeMailPublicFolder     False                           String
Microsoft.OptionalFeature.UserWriteBack                False                           String
Microsoft.Synchronize.StagingMode                      False                           String
Microsoft.OptionalFeature.ExportDeletionThreshold      False                           String
Microsoft.DeviceWriteBack.Forest                                                       String
Microsoft.OptionalFeature.DeviceWriteUp                True                            String
Microsoft.OptionalFeature.HybridExchange               False                           String
Microsoft.AADFilter.ApplicationList                                                    String
Microsoft.DirectoryExtension.SourceTargetAttributesMap                                 String
Microsoft.UserWriteBack.Forest                                                         String
Microsoft.DeviceWriteBack.Container                                                    String
Microsoft.UserWriteBack.Container                                                      String
Microsoft.UserSignIn.SignOnMethod                      PasswordHashSync                String
Microsoft.OptionalFeature.GroupWriteBack               False                           String
#>
```

On a new installation of AAD Connect there are already a lot of Parameters, neat.  I do not recommend messing Parameters named like Microsoft.*, the same is true for messing with the Windows registry (you're asking for trouble).

Can new Parameters be added?  Well yes they can!  Here's how:

```powershell
$globalSettings = Get-ADSyncGlobalSettings                           
$globalSettings.AddOrReplaceConfigurationParameter('fooName', 'fooValue')
Set-ADSyncGlobalSettings -GlobalSettings $globalSettings
<#
Version          : 396
SqlSchemaVersion : 615
InstanceId       : d112f5f4-4248-4959-8dc8-eac88e531433
Schema           : Microsoft.IdentityManagement.PowerShell.ObjectModel.Schema
Parameters       : {Microsoft.Synchronize.SynchronizationPolicy, Microsoft.SynchronizationOption.JoinCriteria, Microsoft.UserSignIn.DesktopSsoEnabled, Microsoft.Synchronize.MaintenanceEnabled...}
#>
```

Note the Version property did not increment.  Calling Get-ADSyncGlobalSettings again shows that the Version actually does increment:

```powershell
Get-ADSyncGlobalSettings
<#
Version          : 397
SqlSchemaVersion : 615
InstanceId       : d112f5f4-4248-4959-8dc8-eac88e531433
Schema           : Microsoft.IdentityManagement.PowerShell.ObjectModel.Schema
Parameters       : {Microsoft.Synchronize.SynchronizationPolicy, Microsoft.SynchronizationOption.JoinCriteria, Microsoft.UserSignIn.DesktopSsoEnabled, Microsoft.Synchronize.MaintenanceEnabled...}
#>
```

To get the Parameter, just call Get-ADSyncGlobalSettings then use the new Parameter name as the index item:

```powershell
$globalSettings = Get-ADSyncGlobalSettings
$globalSettings.Parameters['fooName']
<#
Name                   : fooName
InputType              : String
Scope                  : SynchronizationGlobal
Description            : 
RegexValidationPattern : 
DefaultValue           : 
Value                  : fooValue
Extensible             : False
PageNumber             : 0
Intrinsic              : False
DataType               : String
#>
```

That's it, a nice place to store configuration values for a synchronization service.  I'll be posting later about experiments with global settings and preventing accidental deletions.