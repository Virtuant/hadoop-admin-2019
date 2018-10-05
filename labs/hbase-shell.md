# links for this lab:

https://hortonworks.com/hadoop-tutorial/introduction-apache-hbase-concepts-apache-phoenix-new-backup-restore-utility-hbase/

https://community.hortonworks.com/questions/18552/introduction-of-hbase-namespaces-into-a-pre-existi.html

https://www.cloudera.com/documentation/enterprise/5-6-x/topics/cdh_ig_hbase_shell.html


the first of these links is pretty good. I really could not find anything better on this particular lab.


## Lab: Using the HBase Shell
## Objective:
To become familiar with HBase shell operations

### File locations:

### Successful outcome:
You will:
Explore the structure of HBase Shell and perform some command line HBase operations


### Lab Steps

Complete the setup guide

Explore the structure of HBase Shell and perform some command line HBase operations

----

1. Launch the HBase Shell

    ```console
    # hbase shell
    ```
    
    Some information messages and a prompt will appear that looks like:

    ```console
    2016-12-08 21:35:22,119 INFO	[main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available HBase Shell; enter 'help<RETURN>' for list of supported commands.
    Type "exit<RETURN>" to leave the HBase Shell Version 0.98.4.2.2.4.2-2-hadoop2, rdd8a499345afc1ac49dc5ef212ba64b23abfe110, Tue Dec 8 16:18:12 EDT 2015 
    hbase(main):001:0>
    ```
    
2. Get the version of HBase - issue the command:

    ```console
    hbase(main):001:0> version
    ```

3. Get the status of HBase:

    ```console
    hbase(main):00:0> status
    4 servers, 0 dead, 1.0000 average load
    ```
    
    > Note: this shows there are four regionservers in the HBase cluster.

4.	Display a detailed status of HBase:
 
    ```console
    hbase(main):003:0> status 'detailed' 
    version 0.98.4.2.2.4.2-2-hadoop2
    0 regionsInTransition master coprocessors: []
    4 live servers
    node2:60020 1431017576356
    requestsPerSecond=0.0, numberOfOnlineRegions=2, usedHeapMB=44, maxHeapMB=966, numberOfStores=2, numberOfStorefiles=2, storefileUncompressedSizeMB=0, storefileSizeMB=0, memstoreSizeMB=0, storefileIndexSizeMB=0, readRequestsCount=4, writeRequestsCount=4, rootIndexSizeKB=0, totalStaticIndexSizeKB=0, totalStaticBloomSizeKB=0, totalCompactingKVs=0, currentCompactedKVs=0, compactionProgressPct=NaN, coprocessors=[]
    "reptable,,1431026240303.6abf29adb8caab05546222a5c138c228." numberOfStores=1, numberOfStorefiles=1, storefileUncompressedSizeMB=0, storefileSizeMB=0, memstoreSizeMB=0, storefileIndexSizeMB=0, readRequestsCount=4, writeRequestsCount=3, rootIndexSizeKB=0, totalStaticIndexSizeKB=0, totalStaticBloomSizeKB=0, totalCompactingKVs=0, currentCompactedKVs=0, compactionProgressPct=NaN
    "testtable,,1431030312420.22b097dbaa7810490eb168b1cbe8fedb." numberOfStores=1, numberOfStorefiles=1, storefileUncompressedSizeMB=0, storefileSizeMB=0, memstoreSizeMB=0, storefileIndexSizeMB=0, readRequestsCount=0, writeRequestsCount=1, rootIndexSizeKB=0, totalStaticIndexSizeKB=0, totalStaticBloomSizeKB=0, totalCompactingKVs=0, currentCompactedKVs=0, compactionProgressPct=NaN
    ```

5. Use the whoami command:

    ```console
    hbase(main):00:0> whoami 
    student (auth:SIMPLE)
        groups: root, hadoop, hdfs
    ```

6. Ask HBase for help:

    ```console
    hbase(main):0:0> help
    HBase Shell, version 0.96.0.2.0.6.0-76-hadoop2, re6d7a56f72914d01e55c0478d74e5cfd3778f231, Thu Oct 17 18:15:20 PDT 2016
    ```

7.	Type 'help "COMMAND"' (e.g. 'help "get"' - the quotes are necessary) for help on a specific command. Commands are grouped. Type 'help "COMMAND_GROUP"', (e.g. `help 'general'`) for help on a command group. `ddl` stands for Data Definition Language. So help on creating tables defining column family attributes will be displayed by typing:

    ```console
    help 'ddl' 
    help 'get' 
    help 'put' 
    help 'scan' 
    help 'delete'
    ```
    
8. Create a table:

    ```console
    hbase(main):00:0> create 't1','cf1'
    0 row(s) in 0.5070 seconds
    ```

9.	List the table you just created using the list command:

    ```console
    hbase(main):00:0> list
    ```
    
10. Put some data in the table using the put command:

    ```console
    hbase(main):0:0> put 't1', '1','cf1:name','yourname'
    0 row(s) in 0.1480 seconds
    ```

11. Scan the table using the scan command:
 
    ```console
    hbase(main):0:0> scan 't1'
    ```

12. Add another row:

    ```console
    hbase(main):013:0> put 't1', '2','cf1:name','theRainInSputnik' 
    hbase(main):014:0> scan 't1'
    ```
    
13. Change your name:

    ```console
    hbase(main):015:0> put 't1', '1','cf1:name','your_new_name'
    0 row(s) in 0.0070 seconds
    ```
    
14. Drop the table:

    ```console
    hbase(main):020:0> disable 't1'
    0 row(s) in 1.2840 seconds

    hbase(main):021:0> drop 't1'
    0 row(s) in 0.1630 seconds
    ```
 
15. Create a table that stores more than one version of a column:

    ```console
    hbase(main):0:0> create 't1', {NAME => 'f1', VERSIONS => 2}
    ```

    This creates table t1, with column family f1 and any data stored in column family f1 will be permitted to have up to 2 versions. Versions beyond 2 will be deleted, oldest first.
    
16. Insert multiple versions of a column:

    ```console
    hbase(main):0:0> put 't1','1', 'f1:name','name1'
    hbase(main):04:0> put 't1','1', 'f1:name','name2'
    ```

17. Scan the table requesting multiple versions (note different timestamps):

    ```console
    hbase(main):0:0> scan 't1',{VERSIONS => 2} 
    ROW	COLUMN+CELL
    1	column=f1:name, timestamp=1390167231632, value=name2
    1	column=f1:name, timestamp=1390167226238, value=name1
    ```
    
18. Add a third value for the column identifier `f1:name`:

    ```console 
    hbase(main):0:0> put 't1','1', 'f1:name','name3'
    0 row(s) in 0.0080 seconds
    ```
    
19. Scan again:

    ```console 
    hbase(main):0:0> scan 't1',{VERSIONS => 2} 
    ROW	COLUMN+CELL
    1	column=f1:name, timestamp=1390167445021, value=name3
    1	column=f1:name, timestamp=1390167231632, value=name2
    1 row(s) in 0.0110 seconds
    ```

20. Scan for 3 versions:

     ```console 
    hbase(main):0:0> scan 't1',{VERSIONS => 3} 
    ROW	COLUMN+CELL
    1	column=f1:name, timestamp=1390167445021, value=name3
    1	column=f1:name, timestamp=1390167231632, value=name2
    1 row(s) in 0.0170 seconds
    ```

    Get vs. Scan: If the table is large, the scan operation uses a lot of resources. Hbase was designed for the optimal lookup to be a single row get. The syntax shorthand for retrieving all data for a particular row is:

    ```console 
    hbase(main):0:0> get 'table_name', rowkey
    ```

    Specifying `a:name` states that we want the name column in column family "a".

### Result:

You have now explored the structure of HBase Shell and performed some command line HBase operations.
