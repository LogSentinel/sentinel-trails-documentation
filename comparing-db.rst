Comparing Database State
========================
In some cases you may want to verify that the state of your database is not tampered with by comparing it to the logs. In order to do that, you have to be able to match log entries to database records. You can do that in several ways:

* when sending an event, store its ID in your database in a designated column
* pass
* .. code:: text

 entityId


* for each request. The entityId is the ID of your record

Note that if you perform updates, multiple log entries will be linked to the same record. Note: you can fetch all entries regarding a single entity by using.. code:: text

 /api/search/entityHistory

When you want to perform verification, you can fetch records in bulk using the.. code:: text

 /api/search/batch

endpoint by passing your IDs as.. code:: text

 values

.

If you pass log entry IDs (the one LogSentinel returned), you should specify.. code:: text

 field=ID

. If you want to pass.. code:: text

 entityId

, then pass.. code:: text

 field=ENTITY_ID

and.. code:: text

 entityType=YourEntityType

.

You can pass up to 2000 IDs and get the resulting entities. Then you can compare the returned entities to the contents of your database. Again note that when searching by entityId you may get multiple log entries.

Since comparison may be expensive, you should not perform it excessively. Comparing your most recent data once every few days and your entire database every month should be sufficient in most cases.

In case of large amounts of logically immutable data, additional aggregation steps may be taken. After an initial verification using the above approach, a large chunk of records can be hashed together to form a single hash (of e.g. 100 000 records). Then that hash can be pushed to LogSentinel and stored in a separate column next to the other data of each record. Then, a scheduled job (e.g. once a month) could go and recalculate the hash for these records and search LogSentinel for that hash. That way if an attacker has tampered with the database, 1. the hash won’t match even internally, or 2. the hash won’t be found in LogSentinel if the attacker has also recalculated the hash. That way periodic calls over-the-wire to LogSentinel won’t be as heavy, as only a limited amount of data will be transferred.
