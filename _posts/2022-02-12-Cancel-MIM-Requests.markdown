---
layout: single
classes: wide
share: false
title: Cancel a MIM Request
tags: [MIM,PowerShell,LithNet]
---

Sometimes you just want to cancel a bunch of MIM Requests.  You can do it in the MIM Portal using the handy 'Cancel' button when viewing Requests.  

With almost everything in the MIM Portal it is pretty easy to see how that operation is performed by looking at the MIM Request that gets submitted.  In this case I cancelled a Request then looked at the MIM Request history in the MIM Portal.  Turns out that to cancel a Request, you have to submit a new Request (one more request oughta do it).  I put in the GUID of that request then looked at it in PowerShell.  

From the output we can see that changing the 'RequestControl' attribute to 'CancelOperation' tells MIM to cancel the Request (just like operational attributes in LDAP I guess).

Below is an example of how to cancel Requests with script.  LithNet does a really nice job of making it easy!

```powershell
#Query for Request objects
$requests = Search-Resources -XPath "/Request[RequestStatus='PostProcessing']"

#Cancel just one Request
$request[0].RequestControl = 'CancelOperation'
Save-Resource -Resources $request[0]

#View the Request again
Get-Resource -ID $request[0].ObjectID
```

The above sample may not work if the request has active Action Workflow Instances.  In that case the request status will be *PostProcessing* then set the RequestControl property to *CancelActionWorkflow*:

```powershell
#Query for Request objects
$requests = Search-Resources -XPath "/Request[RequestStatus='PostProcessing']"

#Cancel just one Request
$request[0].RequestControl = 'CancelActionWorkflow'
Save-Resource -Resources $request[0]

#View the Request again
Get-Resource -ID $request[0].ObjectID
```

