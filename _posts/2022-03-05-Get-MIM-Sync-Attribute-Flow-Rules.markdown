---
layout: single
classes: wide
share: false
title: Get MIM Sync Attribute Flow
tags: [MIM,PowerShell]
---

MIM Synchronization configurations can get quite large sometimes, making it very difficult to visually navigate configuration such as attribute flow rules.  Recently I needed to share the attribute flow rules from one of our systems, and knew I had done that before so looked for a way to share it.

The [MIMDSC module](https://github.com/Microsoft/MIMDSC/wiki) was created a few years ago to share some work I had done to leverage PowerShell Desired State Configuration for auditing and correcting MIM configuration.  My motivation for completing the MIM Service side of the project waned because it never got high enough in our backlog to warrant the time, but there are some useful nuggets in there (the topic of this post).

# Attribute Flow Rule Challenges
Attribute Flow Rules in MIM Synchronization can be viewed in the Synchronization Manager application, but the user experience is not great when there are lots of rules to navigate through.  The user experience also hides attribute flow rules created in the MIM Portal (declarative synchronization rules).

# Attribute Flow Source of Truth
Synchronization configuration has always been available as giant XML files.  Turns out those XML files provide all the detail in one place, and are relatively easy to read.  That XML can be generated using the svrexport.exe utilitly, also conveniently wrapped using the [Get-MimSyncServerXml](https://github.com/microsoft/MIMDSC/blob/master/Functions/Get-MimSyncServerXml.ps1) function.

# Using MIMDSC to Get Attribute Flow Rules
The [MIMDSC module](https://github.com/Microsoft/MIMDSC/wiki) is now available in the [PowerShell Gallery](https://www.powershellgallery.com/packages/MimDsc/0.0.1).  This script same shows how to use it to get Import Attribute Flow Rules.  Note that the script sample works because a synchronization configuration is included in the module for test purposes, handy eh!

``` powershell
# Install the module to CurrentUser scope
Install-Module mimdsc -Verbose -Scope CurrentUser

# Import the module
Import-Module MIMDSC -Verbose

# Get the import attribute flow
Get-MimSyncImportAttributeFlow -ServerConfigurationFolder (Join-Path $HOME \Documents\WindowsPowerShell\Modules\MimDsc\0.0.1\Tests\MimSyncServerConfiguration) | Format-Table
<#
ID                                     RuleType         MAName  CDObjectType SrcAttribute                          MVObjectType MVAttribute         ScriptContext                                           PrecedenceType PrecedenceRank
--                                     --------         ------  ------------ ------------                          ------------ -----------         -------------                                           -------------- --------------
{1E6AE212-C542-4355-9A45-29886CE561F8} direct-mapping   TinyHR  person       FirstName                             SyncObject   FirstName                                                                   ranked                      1
{533D5321-6941-407F-A5F9-CCD705F72DC8} direct-mapping   TinyHR  person       Initial                               SyncObject   Initials                                                                    ranked                      1
{9997656D-B201-4B38-8504-87724F98CB62} direct-mapping   TinyHR  person       JobTitle                              SyncObject   Title                                                                       ranked                      1
{0022BB53-2436-4C67-ABC2-46024AB4EA21} direct-mapping   TinyHR  person       LastName                              SyncObject   LastName                                                                    ranked                      1
{AB5630B5-3B4B-424B-B783-109AC4F293A3} direct-mapping   TinyHR  person       UserID                                SyncObject   SamAccountName                                                              ranked                      1
{C5681D2C-33D0-47EB-812E-9A90E5B691CC} scripted-mapping TinyHR  person       {FirstName, Initial, LastName, Title} SyncObject   DisplayName         cd.person:FirstName,LastName->mv.SyncObject:DisplayName ranked                      1
{81E84777-4E79-47D8-A967-155FA3057C17} direct-mapping   TinyHR  person       dn                                    SyncObject   Fax                                                                         ranked                      1
{1A6C290A-9982-4111-A2D7-F1FDBE4B2283} constant-mapping TinyHR  person                                             SyncObject   OnPremiseObjectType                                                         ranked                      1
{08CC44AA-55FC-4C60-9A90-28BE4708BDC2} dn-part-mapping  TinyHR  person                                             SyncObject   RetentionUrl                                                                ranked                      1
{86E75ABE-B15D-4CFF-A785-BC968965A361} direct-mapping   GrandHR robot        FirstName                             Contact      DisplayName                                                                 ranked                      1
{385176D8-4EB8-4900-B627-DBAF137C8FF3} scripted-mapping TinyHR  person       {FirstName, LastName}                 Contact      DisplayName         cd.person:FirstName,LastName->mv.Contact:DisplayName    ranked                      2
{AF5CA310-1D8C-4669-95C2-1F7D0482CB8F} direct-mapping   TinyHR  person       LastName                              Contact      DisplayName                                                                 ranked                      3
{E1261AAA-DE1A-4AF0-8373-2C6C6FB76713} direct-mapping   TinyHR  person       FirstName                             Contact      DisplayName                                                                 ranked                      4
{2E6E105E-0C88-45EE-BB15-83C570E8CD3C} direct-mapping   GrandHR robot        LastName                              Contact      LastName                                                                    ranked                      1

#>
```

# Feedback
Pretty sure I'm the only person using PowerShell to play with synchronization configuration.  If anyone else out there is doing the same I'd love to hear about it!
 
