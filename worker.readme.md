Hoodie Shares Plugin / Worker
=============================

**WORK IN PROGRESS**

This worker handles sharing of objects between users or publicly.



What happens behind the curtain
---------------------------------

Here's an example (simplified) `$share` object created by a user:

```json
{
  "_id"  : "$share/uuid567",
  "_ref" : "1-bl2xa#1346886508617",
  "type" : "$share"
}
```

The worker picks it up, creates a database "share/uuid567" and a
continuous replication from the user's database, who created the share.

Whenever the user adds an object to the share, the share id will be
set at the $sharedId attribute

```json
{
  "_id"       : "todo/abc4567",
  "type"      : "todo",
  "name"      : "Remeber the mild",
  "owner"     : "joe@example.com",
  "$sharedId" : "uuid567"
}
```

Whenever the user removes an object from a sharing, an `$unshared : true`
property gets added, so that the worker can react on it and remove the object
from the $shares database. Once the object has been removed, both the `$unshared`
and the `$sharedId` attributes will get removed


The shares database
---------------------

When a user creates a new share, the database gets not created directly.
Instead, the $share object gets copied to the shares database. Once it
was copied successfully, the actuall database gets created. Same with
removing shares.

This has two benefits

1. It provides a central place to follow all activity regarding shares
2. Shares can be managed by directly interacting with the shares database,
   e.g. as admin from pocket.

It also helps with debugging, wich should not be underestimate, as the
shares worker is pretty complex.

The shares database does not only hold $share objects, but also $subscription
objects, that represent replications. They work the same way as middle man
as the $share objects, for the same reason.


The access setting
--------------------

the `access` setting
the access setting defines who can read and/or write to the sharing. Default
value is false, meaning only the creator has access. `true` means the sharing
is public. More granular settings are possible as well:

`{read: true}` public sharing, but read only
`{read: ["user1", "user2"]}` private sharing, only user1 & user2 have read access
`{write: ["user1", "user2"]}` private sharing, user1 & user2 have read & write access
`{read: true, write: ["user1"]}` private sharing, but only user1 has write access

depending on the access setting, a _design doc has to be created that prevents
unauthorized users to make changes to the shared objects. And if the share allows
changes, they need to be replicated to users $shares database and the changes need
to be incorporated into the "real objects".



## To be done

* The current implementation is not bidirectional yet.
  I can read and subscribe to a share, but not make changes yet.


