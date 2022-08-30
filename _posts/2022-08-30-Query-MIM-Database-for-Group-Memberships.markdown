---
layout: single
classes: wide
share: false
title: Query the MIM Service Database for Group Membership
tags: [MIM]
---

Got a MIM Service database backup?  You should.  Got it configured in Azure with a retention policy?  Got it encrypted?

Ran into an issue with MIM Hybrid Reporting a couple years ago and came to the conclusion that the MIM Service database is the ultimate source for MIM audit data because it contains the MIM Service request history.

To guard against losing that we can configure backup retention for a long period.

With that many backups of the MIM Service database it also becomes possible to query the database to see what objects looked like at the time of the backup.  This has come in handy when investigating incidents, to see what an object looked like at different points in time.

With just a database backup you could restore the MIM Service and point it at the database but it is sometimes easier to just query the data directly from SQL.

Here is an example that gets group membership for a user given the ObjectID of their Person object:

_WARNING_ do not do this in production as it could interfere with the MIM Service

``` sql

SELECT 
	OVSGroupAccountName.ValueString AS GroupAccountName,
	OVSGroupDisplayName.ValueString AS GroupDisplayName	
FROM fim.objects AS O
JOIN fim.ObjectValueReference AS OVR ON OVR.ObjectKey = O.ObjectKey AND OVR.AttributeKey = 40 -- 40 is ComputedMember
JOIN fim.Objects AS MO ON OVR.ValueReference = MO.ObjectKey
JOIN fim.ObjectValueString AS OVSGroupAccountName ON O.ObjectKey = OVSGroupAccountName.ObjectKey AND OVSGroupAccountName.AttributeKey = 1  -- 1 is AccountName
JOIN fim.ObjectValueString AS OVSGroupDisplayName ON O.ObjectKey = OVSGroupDisplayName.ObjectKey AND OVSGroupDisplayName.AttributeKey = 66  -- 66 is DisplayName
WHERE MO.ObjectID = 'A0A09E7A-4AE8-4E70-81C5-8008503ED2A3'

/*
GroupAccountName	GroupDisplayName
endo                    Mountain Bike Trail Hikers
foo                     The Fight is Real
bar                     Self Pacification
*/
```
