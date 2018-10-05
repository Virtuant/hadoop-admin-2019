## Lab: Export with Pig and Import with ImportTsv

**Objective**: To connect to an HBase table via Pig and export some data into tab-delimited format; then import that 
data into HBase using ImportTSV. 

An exported tab-delimited file which is then used to update the 'c' column via an import to the users table

----

### In the Shell, do:
    
Scan and describe the users table. You should have a column family 'a' with one version being kept. 
The scan shows that we have:

    ```console
    address, city, email, name, phone, state and zipcode
    ```
    
Now exit the HBase shell.

### Now in Putty:

Open a terminal window and enter the Pig grunt shell:

    ```console
    pig 
    grunt>
    ```

Load the users table and choose everything in column family `a` and use the same names as the table has 
with all being of `charray` type. (Command is all on one line.)

The Pig schema requires an `ID` (rowkey) but we are not passing an ID to the command, so Pig makes one up: 
`ID:bytearray`. Don't forget to include the rowkey.

    ```console
    grunt> x = LOAD 'hbase://users' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage( 'a:address a:city 
    a:email a:name a:phone a:state a:zipcode','-loadKey true') AS (ID:bytearray , address:chararray , 
    city:chararray, email:chararray, name:chararray, phone:chararray, state:chararray, zipcode:chararray);
    ```

And dump the result to the screen:
    
    ```console
    grunt> DUMP x;
    ```
    
This command will read data from HBase and display it on the terminal window.

If you get a class not found TableSplit error, try the following:

    ```console
    sudo cp /usr/lib/hbase/lib/hbase-*hadoop2.jar /usr/lib/Hadoop/lib 
    sudo cp /usr/lib/hbase/lib/htrace*.jar /usr/lib/Hadoop/lib
    sudo cp /usr/lib/hbase/lib/protobuf*.jar /usr/lib/Hadoop/lib
    ```
    
Save HBase data in a new HDFS directory: 'PigUsers':

    ```console
    grunt> STORE x into 'PigUsers'
    grunt> quit
    ```

> Note: you actually saved the file to `/users/student/PigUsers` as the system stores data in that directory. 
You can leave off the directories `/users/student`.
	
Check that the new directory 'PigUsers' exists and verify its content

    ```console
    hdfs dfs -ls PigUsers
    hdfs dfs -cat PigUsers/part-m-00000
    ```

The table is now stored in a tab-delimited format
 
Put the 'PigUsers' file from HDFS into the local file system

    ```console
    hdfs dfs -get PigUsers/part-m-00000 PigUsers.csv
    ls -l PigUsers.csv
    ```

Now in the HBase shell add a new column family in `Users` table. Disable the users table, add a column family `c` and then enable again. Do a describe on the table to make sure our `c` family is now there.

Import a local file into HBase using ImportTsv:

    ```console
    hbase org.apache.hadoop.hbase.mapreduce.ImportTsv \
    - Dimporttsv.columns=HBASE_ROW_KEY,c:address.c:city,c:email,c:name,c:phone,c:stat e,c:zipcode users PigUsers
    ```

Wait for this to complete before continuing.

Scan the `users` table. You should see that the `c` column family is now populated via the import. You have now done a direct import of a CSV file into an HBase table.

Create HFiles from an import and then associate those files with the table:

    ```console
    hbase org.apache.hadoop.hbase.mapreduce.ImportTsv\
    - Dimporttsv.columns="HBASE_ROW_KEY,a" \
    - Dimporttsv.bulk.output=hdfs:///user/student/storefileoutput \ 
    users hdfs:///user/student/PigUsers/part-m-00000
    ```

### Summary

You should have exported data from HBase into Pig and stored it in HDFS as a tab-delimited file; 
and then imported that file into an HBase table using ImportTSV.
