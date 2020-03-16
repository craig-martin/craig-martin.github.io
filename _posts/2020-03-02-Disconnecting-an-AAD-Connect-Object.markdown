---
layout: single
classes: wide
title:  "Disconnecting an AAD Connect Object"
date:   2020-03-02 19:20:42 -0800
share: false
tags: [Azure AAD Connect, Synchronization]
---

The synchronization engine uses joins to enable rules on connected objects.  AAD Connect removed the ability to disconnect joined objects, and it has been asked for as feedback:

* [Reinstate Joiner and other MIM Sync features](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/16396447-reinstate-joiner-and-other-mim-sync-features#comments)
* [Azure Ad Connect - Add possibility to disconnect a jobject with custom Rule](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/18837448-azure-ad-connect-add-possibility-to-disconnect-a)
* [Advanced Search on Joiner Tool in Sync Engine](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/18296323-advanced-search-on-joiner-tool-in-sync-engine)
* [Metaverse Object Properties](https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/11614392-metaverse-object-properties)

Sometimes objects will join with the best of intentions but have unintended consequences, my company seems to be an infinitely renewable supply of weird data.  Correcting bad joins unfortunately requires manual intervention, which used to be as simple as disconnecting an object and joining it to its correct match.

AAD Connect does not have the disconnect feature, so the workaround is to:
* Export the offending Connector Space object using csexport.exe
* Delete the offending Connector Space object using csdelete.exe
* Create an import file using the file contents from Step 1
* Import the file from Step 3 using an import run profile and the 'resume from existing log file' option

The process is nicely captured in this blog post:
[Disconnecting Objects with AADConnect](https://dloder.blogspot.com/2018/11/disconnecting-objects-with-aadconnect.html)

In the past week I've had to do this a few times, almost enough to automate it.  If it comes to that I'll be posting about it again.