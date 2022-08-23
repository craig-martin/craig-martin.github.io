---
layout: single
classes: wide
share: false
title: Get AAD Deleted Users Using Graph
tags: [AAD,Graph]
---

Need to gather detail on deleted AAD user objects, and Graph once again comes to the rescue.

Really happy with how well these APIs work, but real kudos goes to the docs:
- [List deletedItems](https://docs.microsoft.com/en-us/graph/api/directory-deleteditems-list?view=graph-rest-1.0&tabs=http)
- [Advanced Query Capabilities](https://docs.microsoft.com/en-us/graph/aad-advanced-queries?tabs=http#user-properties)

Though the API is well documented there was a slight twist that took me a few minutes to figure out.

```
https://graph.microsoft.com/v1.0/directory/deletedItems/microsoft.graph.user?$filter=endswith(mail,'@litware.ca')&$count=true
```

``` json
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
    "@odata.count": 2,
    "value": [
        {
            "displayName": "Nino Schurter",
            "givenName": "Nino",
            "mail": "nino@litware.ca",
            "officeLocation": "Van Life"
        },
        {
            "displayName": "Jenny Risveds",
            "givenName": "Jenny",
            "mail": "Jenny@litware.ca",
            "surname": "Risveds"
        }
    ]
}
```

In my case, two issues prevented this from working:
- leaving out $count from the query string 
- leaving out the ConsistencyLevel header with a value of 'eventual' 

The sample above worked great, here is what I ended up with to:
- return only Litware.ca users
- return only items deleted in the past couple of days (we delete a lot of users)
- return the count of deleted users
- return just a few attributes for each user

```
https://graph.microsoft.com/v1.0/directory/deletedItems/microsoft.graph.user?$filter=endswith(mail,'@litware.ca')+and+deletedDateTime+ge+2022-08-20T00:00:00Z&$count=true&$select=mail,mailnickname,employeeid,onPremisesImmutableId
```
