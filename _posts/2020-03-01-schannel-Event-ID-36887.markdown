---
layout: single
classes: wide
title:  "Schannel Event ID 36887 - fatal alert was received: 46"
date:   2020-03-01 19:20:42 -0800
share: false
tags: [TLS, Debugging, Security]
---
Did battle with this event log error recently.  It turned out to be an IIS binding configuration where Server Name Indication (SNI) was turned off.  Since SNI was not enabled for the web application,  IIS would accept the request which then failed the SSL handshake, producing the schannel error.  Turning on SNI (requiring SNI) tells IIS to refuse the HTTPS request to the IP address, so the client will get an error and will need to use the hostname.

To reproduce:
* Turn off SNI in the IIS web application bindings
* Browse to the site using ```HTTPS://<IP Address>```
* *FAIL*
* See Event ID 36887 in the Event Log showing an error code of 46