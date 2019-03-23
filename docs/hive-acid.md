## Interactive Query with Hive and Tez

**Objective**: We will learn to use Hadoop as a system of record.

**Exercise directory**: `~/data`

Hadoop is gradually playing a larger role as a system of record for many workloads. Systems of record need robust and varied options for data updates that may range from single records to complex multi-step transactions.

Some reasons to perform updates may include:

* Data restatements from upstream data providers.
* Data pipeline reprocessing.
* Slowly-changing dimensions (e.g. SCD Type 1)
* Dimension history / evolution (e.g. SCD Type 2)

Standard SQL provides ACID operations through INSERT, UPDATE, DELETE, transactions, and the more recent MERGE operations. These have proven to be robust and flexible enough for most workloads. Hive offers INSERT, UPDATE and DELETE, with more of capabilities on the roadmap.

#### Concepts

* Transactional Tables: Hive supports single-table transactions. Tables must be marked as transactional in order to support UPDATE and DELETE operations.
* Partitioned Tables: Hive supports table partitioning as a means of separating data for faster writes and queries. Partitions are independent of ACID. Large tables in Hive are almost always partitioned. Large ACID tables should be partitioned for optimal performance.
* ACID Operations (INSERT / UPDATE / DELETE): Standard SQL commands that allow data inserts, updates and deletes.
* Primary Key: Databases use primary keys to make records easy to locate, which facilitates updates or deletes. Hive does not enforce the notion of primary keys, but if you plan to do large-scale updates and deletes you should establish a primary key convention within your application.
* Streaming Ingest: Data can be streamed into transactional Hive tables in real-time using Storm, Flume or a lower-level direct API.
* Optimistic Concurrency: ACID updates and deletes to Hive tables are resolved by letting the first committer win. This happens at the partition level, or at the table level for unpartitioned tables.
* Compactions: Data must be periodically compacted to save space and optimize data access. It is best to let the system handle these automatically, but these can also be scheduled in an external scheduler.

----

Hello ACID: Create a Partitioned ACID Table and Insert some Data

    Log in to Ambari using user credentials maria_dev/maria_dev.

    Go to Data Analytics Studio or DAS and click on the Data Analytics Studio UI or go to port sandbox-hdp.hortonworks.com:30800.

Let’s start by creating a transactional table. Only transactional tables can support updates and deletes. Within the Hive View query editor insert this query text:

```sql
DROP TABLE IF EXISTS hello_acid;

CREATE TABLE hello_acid (key int, value int)
PARTITIONED BY (load_date date)
CLUSTERED BY(key) INTO 3 BUCKETS
STORED AS ORC TBLPROPERTIES ('transactional'='true');
```

Within the DAS it will look as below. Press Execute to create the table.

![create-table-hello-acid-800x310](https://user-images.githubusercontent.com/558905/54872624-45eee880-4d9d-11e9-9b8c-981b66228899.jpg)

From here on out, everything is familiar SQL you’ve likely used for many years. Here is an example that inserts some records, deletes one record and updates one record.

```sql
INSERT INTO hello_acid partition (load_date='2016-03-01') VALUES (1, 1);
INSERT INTO hello_acid partition (load_date='2016-03-02') VALUES (2, 2);
INSERT INTO hello_acid partition (load_date='2016-03-03') VALUES (3, 3);
SELECT * FROM hello_acid;
```

You should see the following results:

![insert-into-hello-acid-800x476](https://user-images.githubusercontent.com/558905/54872625-45eee880-4d9d-11e9-8241-2979e0bad7d4.jpg)

Next, let’s delete and update data in the same window execution:

DELETE FROM hello_acid WHERE key = 2;
UPDATE hello_acid SET value = 10 WHERE key = 3;
SELECT * FROM hello_acid;

You should see this:

![update-hello-acid-800x457](https://user-images.githubusercontent.com/558905/54872628-45eee880-4d9d-11e9-8ccb-1f59b2c06971.jpg)

This example shows the most basic ways to add data into a Hive table using INSERT, UPDATE and DELETE commands. Later we will see some more powerful ways of adding data to an ACID table that involve loading staging tables and using INSERT, UPDATE or DELETE commands, combined with subqueries, to manage data in bulk.

These DML commands are designed to deal with large amounts of data in a microbatch manner. These DML statements should not be used for record-level data management. If you have small batches of constantly arriving, you should use Streaming Data Ingestion instead.
Practical Uses of SQL ACID

Inserting a couple of records helps to get acquainted but in a real setting you need to deal with thousands or millions of records at a time. This section discusses how to get deal with data batches across a number of common scenarios.

These patterns require you to establish a primary key. Hive does not enforce primary key uniqueness, you will need to do this in your application. Although Hive 2.1 introduced the notion of non-validating foreign key relationships. There are currently no integrity checks enforced by the system.

### Searched Updates

Hive ACID supports searched updates, which are the most typical form of updates. It is important to realize that, based on Hive ACID’s architecture, updates must be done in bulk. Doing row-at-a-time updates will not work at any practical scale. Searched updates can work well when you’re doing a first pass to update a large number of records in a fixed way.

For example let’s consider a dimension table which includes a flag to indicate whether the record is the most current value. This allows tracking a dimension’s evolution over time, a common strategy for dealing with slowly-changing dimensions (SCDs). When a set of dimension table updates arrive you want to mark all existing values as old. This is easy to do with an in list and subquery on a common key.

```sql
DROP TABLE IF EXISTS mydim;

CREATE TABLE mydim (key int, name string, zip string, is_current boolean)
CLUSTERED BY (key) INTO 3 BUCKETS
STORED AS ORC TBLPROPERTIES ('transactional'='true');

INSERT INTO mydim VALUES
  (1, 'bob',  '95136', true),
  (2, 'joe',  '70068', true),
  (3, 'steve', '22150', true);

DROP TABLE IF EXISTS updates_staging_table;

CREATE TABLE updates_staging_table (key int, newzip string);

INSERT INTO updates_staging_table VALUES (1, 87102), (3, 45220);
```

— Before

```sql
SELECT * FROM mydim;

UPDATE mydim SET is_current = false
  WHERE mydim.key IN (SELECT key FROM updates_staging_table);
```

— After

```sql
SELECT * FROM mydim;
```

The result will be that records 1 and 3 have their is_current flag set to false. From there we can add the new, updated values to the end of the table with their is_current flag set to true.

### Searched Deletes

Bulk deletes can be done easily with a staging table and in list + subquery. This requires you have a common key between the tables, similar to how you would use a primary key in RDBMS.

Example:

```sql
DELETE FROM mydim
  WHERE mydim.key IN (SELECT key FROM updates_staging_table);

SELECT * FROM mydim;
```

### Overwriting Existing Records with New Records

There are situations where you need to update a batch of records to a new set of values. For example, Type 1 SCD updates or restatements of inaccurate data. Hive now supports SQL MERGE, which will make this task easy.

### Operational Tools for ACID

ACID transactions create a number of locks during the course of their operation. Transactions and their locks can be viewed using a number of tools within Hive.
Seeing Transactions:

```sql
show transactions;
```

This command shows active and aborted transactions. Here is some example output:

![show_transactions-800x319](https://user-images.githubusercontent.com/558905/54872627-45eee880-4d9d-11e9-9d1b-1d5175738089.png)

Seeing Locks:

```sql
show locks;
```

This command shows locks, along with their associated transaction IDs. Example:

![show_locks-800x142](https://user-images.githubusercontent.com/558905/54872626-45eee880-4d9d-11e9-8ff0-7e3c3a4c819a.png)

Locks can be Read, Update or X locks. Update lock is compatible with Read locks but not other updates. X is not compatible with anything.

### Aborting Transactions:

```sql
ABORT TRANSACTIONS transactionID;
```

It may be necessary to abort a transaction, for example because a transaction is running too long. You can abort a set of transactions using “abort transactions” followed by a list of numeric transaction IDs. Note that aborting a transaction won’t kill the related query immediately. Instead ACID queries periodically heartbeat, every 2.5 minutes by default, and if they detect their underlying transaction has been aborted they will exit at that time.
Real-World Performance Considerations

These performance tips will help you survive in the real world:

1. Partition your data. Hive compactions are not tiered: major compactions re-write all data in modified partitions, one partition at a time. Partitioning data is essential to ensure you can manage large datasets without degradation. Partitioning by date is the most common approach.
2. Inserts are fast, updates and deletes are relatively slower. Updates and deletes perform full partition scans. Plan for this by batching data appropriately.
3. If your workload includes a large number of updates and deletes, compact regularly. Your total data size will grow until you compact, and analytical queries will slowly degrade until compaction is done.

### For Experts Only

In the real world things go wrong. When things go wrong you need options for creative solutions. At Hortonworks we have used the information in this section to get past some very tricky problems. Warning: Improper application of this information may cause data corruption or permanent data loss. Ensure you fully understand the system before using this information, test it out on data you can afford to lose and always backup any data you really care about.


> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) ACID tables have a hidden column called `row__id`. You should consider this column a system internal and assume that its name or even its presence may change at any time without warning. You should not build a long-term solution on top of this column, only use it to get you past a tough spot.

With that out of the way, this column records:

* The transactionid that was active when the data was inserted or updated.
* The bucketid, the bucket number where the data lives.
* The rowid, the rowid within this transaction/bucket combo.

You can access this data as follows using beeline:

```console
beeline -u jdbc:hive2://sandbox-hdp.hortonworks.com:10000 -n hive

0: jdbc:hive2://sandbox.hortonworks.com:2181/> select row__id from hello_acid;
```

Ouptut should look like:

```console
+-----------------------------------------------+
|                    row__id                    |
+-----------------------------------------------+
| {"writeid":1,"bucketid":536936448,"rowid":0}  |
| {"writeid":5,"bucketid":537001984,"rowid":0}  |
+-----------------------------------------------+
```

A common need is to confirm that all records were ingested. Let’s say your upstream provider insists data is missing in Hive. Your provider (e.g. Storm Bolt) can tell you the transaction ID used to insert data. You can count the actual records using the transactionid. Replace X with your transactionid:

```sql
set hive.optimize.ppd=false;
select count(*) from hello_acid where row__id.writeid = {writeid};
```

Keep in mind that data from this transaction may have been deleted by a subsequent UPDATE or DELETE statement, so if the counts don’t match, consider if records may be altered some other way.

Use information related to this hidden field very carefully.

You can exit beeline by issuing the command:

```sql
!quit
```

### Results

You learned how to create Hive tables from .csv files using DAS. We also experimented with MapReduce and Tez to observe the speed improvements of Hive on Tez.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>