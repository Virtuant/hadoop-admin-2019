## Bending time in HBase

From https://www.ngdata.com/bending-time-in-hbase/

HBase, like BigTable, “is a sparse, distributed, persistent multidimensional sorted map. The map is indexed by a row key, column key, and a timestamp; each value in the map is an uninterpreted array of bytes.” (ref) For completeness, we could consider “table” and “column family” as other dimensions offered by HBase.

The dimensions are not all equal: they each have their own behavior. For example, the row dimension is the one that gets sharded, so can grow very big. Columns are not sharded, but in contrast to rows, multiple columns within a row can be atomically put or deleted (fine print: doing puts and deletes as one atomic operation is not possible). In this article, we will focus on the specifics of the time dimension.

You could consider the time dimension to be some automatic versioning done by HBase, which you can ignore if you do not need it. Indeed, you can do operations like get, put or delete without specifying a time, and it will usually work like you would expect. However, some particularities of the current implementation make that you are better of having a good understanding of the time dimension anyway.
Basics
Terminology

In the BigTable paper, the terminology used is that a {row key, column key} pair addresses a cell, and that each cell can contain multipleversions, indexed by timestamp.
Keys in the time/version dimension

While rows keys and column keys (a.k.a. qualifiers) are bytes, in the time dimension, the key is a long integer. Typically this contains time instants like the ones returned by java.util.Date.getTime() or System.currentTimeMillis(), that is: “the difference, measured in milliseconds, between the current time and midnight, January 1, 1970 UTC”.

The time dimension is stored in decreasing order, so that when reading from a store file, the most recent values are found first.

We will now look at the behavior of the time dimension for each of the core operations: get, put and delete.
Get

By default, when doing a get, the version with the biggest timestamp of each cell is returned (which may or may not be the latest one written, see later).

The default behavior can be changed in two ways, you can ask:

    to return more than one version: see Get.setMaxVersions()
    to return other versions than the latest ones, see Get.setTimeRange()

One interesting option that is missing is the ability to retrieve the latest version less than or equal to a given timestamp, thus giving the ‘latest’ state of the record at a certain point in time. Update: this is (obviously) possible: just use a range from 0 to the desired timestamp and set the max versions to 1 (thanks Jonathan).
Put

Doing a put always creates a new version of a cell, at a certain timestamp. By default the system uses currentTimeMillis, but you can specify the timestamp (= the long integer) yourself, on a per-column level. This means you could assign a time in the past or the future, or use the long value for non-time purposes.

Without going too much into the storage architecture, HBase basically never overwrites data but only appends. The data files are rewritten once in while by a compaction process. A data file is basically a list of key-value pairs, where the key is the composite {row key, column key, time}. Each time you do a put that writes a new value for an existing cell, a new key-value pair gets appended to the store. Even if you would specify an existing timestamp. Doing lots of updates to the same row in a short time span will lead to a lot of key-value pairs being present in the store. Depending on the garbage collection settings (see next), these will be removed during the next compaction.
Delete

There is a lot to tell about deletes and the time dimension.

Garbage collection

First of all, there are two ways of automatic pruning of versions:

    you can specify a maximum number of versions, if more versions are added the oldest ones are deleted. The default is 3, and is configured when creating a column family via HColumnDescriptor.setMaxVersions(int versions). The actual deletion of the excess versions is done upon major compaction, though when performing gets or scans the results will already be limited to the maximum versions configured. Thus setting maximum-versions to 1 does not really disable versioning; each put still creates a new version, but only the latest one is kept.
    you can specify a time-to-live (TTL), if versions get older than this TTL they are deleted. The default TTL is “forever”, and is configured via HColumnDescriptor.setTimeToLive(int seconds). Again, the actual removal of versions is done upon major compaction, but gets and scans will stop returning versions whose TTL is passed immediately. Note that when the TTL has passed for all cells in a row, the row ceases to exist (HBase has no explicit create or delete of a row: it exists if there are cells with values in them).

An interesting behavior I noticed while empirically verifying these behaviors is the following: suppose you create three cell versions at t1, t2 and t3, with a maximum-versions setting of 2. So when getting all versions, only the values at t2 and t3 will be returned. But if you delete the version at t2 or t3, the one at t1 will appear again. Obviously, once a major compaction has run, such behavior will not be the case anymore (thus major compactions are not completely transparent to the user).

Manual delete

When performing a delete operation in HBase, there are two ways to specify the versions to be deleted:

    delete all versions older than a certain timestamp
    delete the version at a specific timestamp

A delete can apply to a complete row, a complete column family, or to just one column. It is only in the last case that you can delete versions at a specific timestamp. For the deletion of a row or all the columns within a family, it always works by deleting all cells older than a certain time.

Deletes create tombstone markers

For example, let’s suppose we want to delete a row. For this you can specify a timestamp, or else by default the currentTimeMillis is used. What this means is “delete all cells where the timestamp is less than or equal to this timestamp”. HBase never modifies data in place, so for example a delete will not immediately delete (or mark as deleted) the entries in the storage file that correspond the delete condition. Rather, a so-called “tombstone” is written, which will mask the deleted values. When HBase does a major compaction, the tombstones are processed to actually remove the dead values, together with the tombstones themselves.

If the timestamp you specified when deleting a row is larger than the timestamp of any value in the row, then you can consider the complete row to be deleted.
Uses of timestamps

While the time dimension is primarily intended for versioning, you could consider to use it as a just another dimension similar to columns, with the difference that they key is a long. Due to some bugs, currently this cannot be recommended (see below).

Browsing through Jira issues, I have found some interesting mentions of the timestamp dimension:

    HBase multi data center replication makes use of the timestamps to avoid conflicts. See HBASE-1295 (page 4 of the attached PDF) orthis mail.
    A comment in HBASE-2406 mentions: If you want a consistent version of some data that spans multiple tables (i.e. secondary index), you may want to use the same timestamp to insert into both tables so that you can use the exact timestamp as part of a get() after reading it out of one table.

Limitations

There are still some bugs (or at least ‘undecided behavior’) with the time dimension that needs to be cleared out. These are things that could get you into trouble:

(update: this is solved in HBase 0.90) Overwriting values at existing timestamps. (HBASE-1485 , HBASE-2406) In other words, update the value at an exact {row, column, time} key (I find this the desired behavior, though you could also store multiple unordered values at the same time). It might seem logical that there is trouble here, as you could imagine that HBase needs the timestamp to decide what value is most recent, and if two values have the same timestamp, it can’t make this decision. However, there is another source of order which is the order in which things are written to the HFile.

Deletes mask puts, even puts that happened after the delete(HBASE-2256). Remember that a delete writes a tombstone, which only disappears after then next major compaction. Suppose you do a delete of everything <= T. After this you do a new put with a timestamp <= T. This put, even if it happened after the delete, will be masked by the delete tombstone. Performing the put will not fail, but when you do a get you will notice the put did have no effect. It will start working again after the major compaction has run.

These issues should not be a problem if you use always-increasing timestamps for new puts to a row. But they can occur even if you do not care about time: just do delete and put immediately after each other, and there is some chance they happen within the same millisecond (testimony).
