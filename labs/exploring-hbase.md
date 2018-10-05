## Lab: Exploring HBase

**HDFS paths:** `/hbase/data/default` `/hbase/data/default/user`  `/hbase/data/default/movie`

**HBase tables**:     `movies, user, hbase:meta, tablesplit`

## About This Lab

### Objective:
Use the HBase Shell to explore the `hbase:meta` table.

### Successful outcome:
Have a basic understanfing of HBase shell and meta table.

### Lab Steps

In this exercise you will use the HBase Shell to explore the `hbase:meta` table.

If you haven't impoted the data into HBase then do that now.

----

1.  Invoke the HBase shell and print the help menu:

    ```console
    # hbase shell
    ```
    
1.  Get the status of the HBase cluster:

    ```console
    hbase(main):000:0> status 'simple'
    ```

    Results:

    ```console
     1 live servers

     localhost:60020 1425572833316 requestsPerSecond=0.0, numberOfOnlineRegions=4,usedHeapMB=72, maxHeapMB=503, numberOfStores=6, numberOfStorefiles=8, storefileUncompressedSizeMB=34, storefileSizeMB=34, compressionRatio=1.0000, memstoreSizeMB=0, storefileIndexSizeMB=0, readRequestsCount=82856, writeRequestsCount=37055, rootIndexSizeKB=43, totalStaticIndexSizeKB=20, totalStaticBloomSizeKB=32, totalCompactingKVs=33915, currentCompactedKVs=33915, compactionProgressPct=1.0, coprocessors=[]

     0 dead servers

     Aggregate load: 0, regions: 4
    ```

    This command shows the status of all nodes in the cluster. It breaks the information down into live and dead servers. The live servers show information about the load and number of regions each node is handling.

1.  Scan the `hbase:meta` It will give information about the tables that HBase is serving.

    ```console
    hbase> scan 'hbase:meta'
    ```

    Results:

	|ROW|COLUMN+CELL|
	|---|---|
	|hbase:namespace,, |... column=info:regioninfo, timestamp=1425389660428, ...|
	|movie,,|...column=info:regioninfo, timestamp=1425402913683, ...|
	|movie,, ...|column=info:seqnumDuringOpen, timestamp=1425572846014,value=\x00\x00\x00\x00\x00\x00\x00\x0A, ...|
	|movie,, ...|column=info:server, timestamp=1425572846014,value=localhost:60020, ...|
	|movie,, ...|column=info:serverstartcode, timestamp=1425572846014,value=1425572833316, ...|
	|user,, ...|column=info:regioninfo, timestamp=1425402933390, ...|
	|...||

    The `hbase:meta` table contains information about the node that is serving the table. It also keeps track of the start and end keys for the region. Clients use this information to know which node or RegionServer to contact to access a row.

1.  Create a new table:

    ```console
    hbase> create 'tablesplit', 'cf1', 'cf2', {SPLITS => ['A', 'M', 'Z']}
    ```
    
    When creating a new table in HBase, you can split the table into regions as the starting point. The splits passed in during the create will serve as the initial regions for the table.

1.  Get the status of the HBase cluster:

    ```console
    hbase> status 'simple'
    ```

    Results:

    ```console
    1 live servers

    localhost:60020 1425572833316 requestsPerSecond=0.0, numberOfOnlineRegions=8,

    usedHeapMB=71, maxHeapMB=503, numberOfStores=14, numberOfStorefiles=8, storefileUncompressedSizeMB=34, storefileSizeMB=34, compressionRatio=1.0000, memstoreSizeMB=0, storefileIndexSizeMB=0, readRequestsCount=82887, writeRequestsCount=37063, rootIndexSizeKB=43, totalStaticIndexSizeKB=20, totalStaticBloomSizeKB=32, totalCompactingKVs=33915, currentCompactedKVs=33915, compactionProgressPct=1.0, coprocessors=[]

    0 dead servers

    Aggregate load: 0, regions: 8
    ```

    The number of regions increased by four to account for the four regions in the tablesplit table.

1.  Scan the hbase:meta table again:

    ```console
    hbase> scan 'hbase:meta'
    ```

    > Note that the hbase:meta information for the tablesplit table has multiple regions and those regions have start and end row keys. Look at how HBase took the splits in the table creation command and made regions out of them.

1.  Drop the tablesplit table:

    ```console
    hbase> disable 'tablesplit'
    hbase> drop 'tablesplit'
    ```

1.  Flush the movie and user. This writes out the data from the memstore to HDFS:

    ```console
    hbase> flush 'movie'  
    hbase> flush 'user'
    ```

1.  Quit the HBase shell:

    ```console
    hbase> quit
    ```

1.  Get the full path in HDFS where the movie table's info column family's data is stored:

    ```console
    # hdfs dfs -ls /hbase/data/default/movie/*/info
    ```

1.  Using the path from the ls, run the following command and replace the movie path with the previous command's path output:

    ```console
    # hbase hfile --printkv --file hdfs://localhost:8020/<moviepath>
    ```

    Results:

    ```console
    2015-03-05 18:44:00,210 INFO [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
		2015-03-05 18:44:00,580 INFO [main] util.ChecksumType: Checksum using org.apache.hadoop.util.PureJavaCrc32
    2015-03-05 18:44:00,581 INFO [main] util.ChecksumType: Checksum can use org.apache.hadoop.util.PureJavaCrc32C
    2015-03-05 18:44:02,680 INFO [main] Configuration.deprecation:  fs.default.name is deprecated. Instead, use fs.defaultFS
    2015-03-05 18:44:03,056 INFO [main] hfile.CacheConfig: Allocating LruBlockCache with maximum size 396.7 M
    K: 1/info:average/1425593979703/Put/vlen=4/mvcc=0 V: 4.15 
		K: 1/info:count/1425593979703/Put/vlen=4/mvcc=0 V: 2077
    K: 10/info:average/1425593979703/Put/vlen=4/mvcc=0 V: 3.54 
		K: 10/info:count/1425593979703/Put/vlen=3/mvcc=0 V: 888
    K: 100/info:average/1425593979703/Put/vlen=4/mvcc=0 V: 3.06 
		K: 100/info:count/1425593979703/Put/vlen=3/mvcc=0 V: 128
    K: 1000/info:average/1425593979703/Put/vlen=4/mvcc=0 V: 3.05 
		K: 1000/info:count/1425593979703/Put/vlen=2/mvcc=0 V: 20
    K: 1002/info:average/1425593979703/Put/vlen=4/mvcc=0 V: 4.25
		K: 1002/info:count/1425593979703/Put/vlen=1/mvcc=0 V: 8
    K: 1003/info:average/1425593979703/Put/vlen=4/mvcc=0 V: 2.94
    ...
    ```

    The above command allows you to see how HBase stores the HFiles. All row keys are stored in sorted order; and, for each row key, all column descriptors are stored in sorted order.

	> Note: The preceding screen shot shows an output labeled mvcc. HBase maintains ACID semantics using Multiversion Concurrency Control (MVCC). MVCC as implemented in HBase enables updates to occur without impacting readers, and without the need to use read locks. With MVCC, old data is not overwritten. Instead, the old data is marked as obsolete once the new data has been added.

	An mvcc value of zero means that all of the file contents can participate in any ongoing transaction.

	In the case of an ongoing scan or read, a just-flushed update can be added to the ongoing read. However, if the mvcc has a higher value than that of the ongoing read, the update is not sent to the Client.

1.  Get the full path in HDFS where the user table's info column family's data is stored:

    ```console
    # hdfs dfs -ls /hbase/data/default/user/*/info
    ```

1.  Using the path from the ls, run the following command and replace the user path with the previous command's path output:

    ```console
    # hbase hfile --printkv --file hdfs://localhost:8020/<userpath>
    ```

    The above command allows you to see how HBase stores the HFiles. All row keys are stored in sorted order and all Column Descriptors are stored in sorted order.

### Exploring Catalog Tables and User Tables

1.  In Firefox, go to the HBase Master’s web interface. Open the Firefox browser and go to port 60010 on your instance's site.

1.  Under the Tables section, click on the tab for ‘Catalog Tables’, then click on the entry for `hbase:meta`.

    > Note: the information that is shown for `hbase:meta`. It shows which RegionServer is serving the `hbase:meta` The table is not split into more regions because the `hbase:meta` table is never split.

1.  Go back to the previous page and click on the ‘User Tables’ tab.

1.  Recreate the tablesplit table in the HBase shell as shown earlier in the exercise.

    ```console
    hbase> create 'tablesplit', 'cf1', 'cf2', {SPLITS => ['A', 'M', 'Z']}
    ```

1.  In the web interface click on the entry for tablesplit. You might have to refresh the screen in order to see the newly created table listed.

1.  Look at the Table Regions section. Note that since you pre-split the table, the regions for the table are shown here. Looking closer at the regions, notice that each region shows the RegionServer serving that region. Each row also shows the start and stop key for every region.

1.  Finally, the Regions by Region Server section shows which RegionServers are responsible for the various portions of the table’s regions.

1.  Click on the link for the RegionServer to view the RegionServer’s web interface.

1.  The RegionServer’s web interface shows metrics about its current state. It shows a more detailed breakdown of each region’s metrics, and also shows information about the configuration and status of the Block Cache.

1.  Scroll down to the Regions section where you will see the four splits you created for the tablesplit.
