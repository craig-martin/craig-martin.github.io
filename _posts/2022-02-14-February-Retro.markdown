---
layout: single
classes: wide
share: false
title: February Retro
tags: [Retro]
---

Looking back over a sprint to add up all the completed work is such a mental reward.  
Too often I blow right through into the next sprint without stopping.  I gotta do these more often! 

# Uploading SQL Index Statistics to Log Analytics
The main FIMService SQL Server database in our system has some performance degregation that has proven difficulit to pinpoint.  The index statistics look pretty bad despite regularly running a Maintenance Plan that should be optimizing them.  Logs show an increase in SQL Timeout exceptions by the FIM Service, and logs also show an increase in HTTP request response time from the MIM Portal to the MIM Service, but those are both just symptoms.  In search of something closer to the cause I'm adding data, in this case the index statistics for all indexes in the FIMService database.  Those statistics are now uploaded regularly to Log Analytics so we can chart them, and even alert on them once we establish a baseline and threshold.  Look for the extraction script in an upcoming post.

# MIM Service Bug Found
The MIM Service [Enumeration Endpoint](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/forefront-2010/ee652436(v=vs.100)) accepts XPath queries then converts them to SQL queries in order to return results to the caller.  While working on an application that depends on MIM data we discovered that sometimes an XPath query can cause the MIM Service to consume 100% of the computer's CPU.  The workaround is pretty simple, restart the MIM Service.  The mitigation is also simple, stop doing that form of XPath query.  It felt good to get root cause on the issue, and I was a little surprised to find a bug like this in such old software!

# Change Tracking in MIM Sync Run History Uploads
We have been uploading MIM Sync Run History details to Log Analytics for a couple years now, I absolutely love having that data for querying and alerting!  The initial change tracking mechanism started to fail, exposing a design flaw (using Event Log ID to store the Run Number) and called for a better change tracking design.  The new design queries Log Analytics to get the existing Run History, and queries WMI on the sync computer to get the Run History, then only uploads Run History that does not already exist in Log Analytics.  Conceptually pretty simple, and cleaner because it no longer depends on the Event Log.  Granting access for a non-admin account to write to the Event Log was a pain, and we no longer need that privilege, yay!

# Uploading SQL Server Lock Data to Log Analytics
Again looking to add data to assist in diagnosing the MIM performance issues we configured our Log Analytics Workspace to grab the SQL Server Locks data from Performance Monitor counters.  Since the Azure Virtual Machine agent does this with just configuration it was super easy but the data will come in very handy when diagnosing the next performance degradation incident.

# Alert When MIM Portal Cannot Connect to the MIM Service
New mantra, "I want to know when our service is down before our users do".  Along those lines I created an Azure Monitor Alert that fires when the MIM Portal cannot connect to the MIM Service.  Luckily the MIM Portal has nice instrumentation here, it raises an Event Log entry that clearly points to this issue.  

# Alert When the MIM Portal Shows an Error Page
Along the lines of the new mantra, the MIM Portal shows in the Event Log each time it shows the MIM error page to a user.  This is less indicative of an outage unless the volume of these goes way up, so the alert threshold is set rather high to avoid getting spammed.

# Alert When the MIM Service Enumeration Endpoint Exceeds Response Threshold
When we experience service degradation the MIM Service Enumeration Endpoint experiences an increase in HTTP request durations, which makes sense because it is waiting for the MIM Service to return data.  Knowing this we created an Azure Monitor Alert to fire when the Enumeration Endpoint HTTP response times exceed a threshold.  The next time performance degrades I think we are going to be bombarded with these new alerts, and we will have a lot of data providing clues pointing to the root cause.


 

