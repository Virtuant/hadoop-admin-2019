## Min Versions and Time-To-Live

**Objective**: Look deeper into HBase's minimum versioning, and time to live settings.

MIN_VERSIONS and Time-To-Live (TTL) are features important for both data life cycle management and storage management in HBase. You can use MIN_VERSIONS together with TTL to more fully express the logic to apply when purging expired cells. 
For example, they can capture a rule like: “Keep metrics reported by each machine. 
Keep at most the last five metrics, throw away metrics that are more than one month old, 
but ensure that you keep at least one value however old it is.”

In this exercise, you will experiment with MIN_VERSIONS, remembering that:

- MIN_VERSIONS should only be set when TTL is enabled for a given column
- MIN_VERSIONS must be set to a value that is less than or equal to the number of row versions to retain,
 i.e. the column family VERSIONS

----

### Using Time-To-Live

Start the HBase shell.

Create a table with the following characteristics:

Table name: tbl_1
Column Family: cf1

    
Do this like:
    
    ```console
    hbase> create 'tbl_1', 'cf1'
    ```
    
List all tables again to verify table tbl_1 was successfully created:

    ```console
    hbase> list
    ```
    
Describe the table you just created:

    ```console
    hbase> describe 'tbl_1'
    ```
    
Notice that the cf column family shows TTL => FOREVER. The default Time-To-Live is FOREVER, 
meaning that versions of a cell never expire.

Let’s enable expiration processing by setting TTL to 60. This means that a version will be deleted 
60 seconds after you insert it into the table:

    ```console
    hbase> alter 'tbl_1', NAME => 'cf1', TTL => 60
    ```
    
Verify that TTL changed from FOREVER to 60 SECONDS:

    ```console
    hbase> describe 'tbl_1'
    ```
    
Put the first row into the table:

    ```console
    hbase> put 'tbl_1', 'rowkey1', 'cf1:cq1', 'Row 1'
    ```
    
View the row in the table:

    ```console
    hbase> scan 'tbl_1'
    ```
    
Wait at least sixty seconds and issue the scan command again to verify that the row you inserted has been 
deleted because it has expired.

> Note: For future reference, setting TTL to 2147483647 (maximum integer value) will change TTL back to FOREVER


### Using Time-To-Live with MIN_VERSIONS

Next let’s make HBase always retain at least one version of a cell, even if the version has expired.

Create a new table for this part of the exercise, set TTL to 10 seconds and set MIN_VERSIONS to 1. 
Now, even if a version expires it will not be deleted if it is the only remaining version of the cell:

    ```console
    hbase> create 'tbl_2', NAME => 'cf2', TTL => 10, MIN_VERSIONS => 1
    ```
    
Verify that TTL and MIN_VERSIONS are set properly for column family cf2:

    ```console
    hbase> describe 'tbl_2'
    ```
    
Let’s do another scan – the table should have no rows:

    ```console
    hbase> scan 'tbl_2'
    ```
    
Create a row by adding data for `cf2:cq2`:

    ```console
    hbase> put 'tbl_2', 'rowkey2', 'cf2:cq2', 'cf2:cq2 Row 1'
    ```
    
View the row in the table:

    ```console
    hbase> scan 'tbl_2'
    ```
    
Wait more than ten seconds and re-run the scan command to verify that the version has not been deleted even though it has expired. Even though the version has expired, HBase has not deleted it because that would mean falling below our minimum versions threshold of 1.

Once you are satisfied that the version will not be deleted, try altering the table and setting `MIN_VERSIONS` to `0\`. This means you drop the requirement that at least one version be retained at all times (even if it has expired):

    ```console
    hbase> alter 'tbl_2', NAME => 'cf2', MIN_VERSIONS => 0
    ```
    
Scan the table and verify that the expired version is no longer available:

    ```console
    hbase> scan 'tbl_2'
    ```
    
Finally, cleanup the table:

    ```console
    hbase> disable 'tbl_1' 
    hbase> drop 'tbl_1'
    hbase> disable 'tbl_2'  
    hbase> drop 'tbl_2'
    ```

### Results


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>

