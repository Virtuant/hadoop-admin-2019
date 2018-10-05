# links to info for these labs:

https://hortonworks.com/hadoop-tutorial/introduction-apache-hbase-concepts-apache-phoenix-new-backup-restore-utility-hbase/

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_data-access/content/ch_hbase_bar.html

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_data-access/content/ch_hbase_bar.html



## Lab: Administration of HBase

## Objective:
Learn how to perform various administrative tasks using HBase utilities.

### File locations:

### Successful outcome:
YOU WILL:
In this exercise, you will perform various administrative tasks using the HBase utilities.


### Lab Steps

Complete the setup guide

**Data directory**:  `~/data/users_orig.tsv`

**HBase tables**:    `user, userbackup, movie, moviebackup, customer`

In this exercise, you will perform various administrative tasks using the HBase utilities.

----

### View Configuration and Log Files
    
1.  Change the directory to the logs directory for HBase (for example, `/var/log/hbase`).

1.  View the logs directory for HBase.

    The `.log files` are the Log4J output from the daemons. The .out files are the output from standard output saved to a file.

1.  View the log files for the Master and RegionServer.

### Backups

There are various ways to backup data in HBase. Each of those methods has different caveats.

1.  Use the built-in HBase table export program to create a backup of the user table.

    ```console
    # hbase org.apache.hadoop.hbase.mapreduce.Export user userbackup
    ```
    
1.  Verify that a directory to hold the exported data has been created in HDFS. You should see a directory called `userbackup`.

1.  Verify that the exported data has been stored to userbackup in HDFS.

1.  In the HBase shell, disable and drop the user table.

1.  Use the built-in HBase table import program to restore the backup of the user table that you just created.

    ```console
    # hbase org.apache.hadoop.hbase.mapreduce.Import user userbackup
    ```
    
    This command will fail because the table does not exist. The import program does not maintain the metadata about the table.

1.  Re-create the user table with the HBase shell. The user table had three column families named `info, ratings and recommendations`.

1.  Use the built-in HBase table import program to restore the backup of the user table again.

    This command will succeed now that the table metadata has been recreated.

1.  Use the built-in HBase table export program to create a backup of the movie table called `moviebackup`.

1.  Go into hbase shell and create a snapshot of the movie:

    ```console
    hbase> snapshot 'movie', 'Movie_Snapshot'`
    ```
    
1.  From the movie table, delete the entire row having a row key value of 367. Verify the row is deleted.

1.  Update a row with the following characteristics:

    ```console
    Table name: 'movie'
    Row key: '231'
    Column Family: 'info'
    Column Descriptor: 'genres' and value 'Drama'
    ```
    
1.  View the newly changed data.

1.  Restore the snapshot of the movie:

    ```console
    hbase> disable 'movie'
    hbase> restore_snapshot 'Movie_Snapshot'
    hbase> enable 'movie'`
    ```
    
1.  Verify the row from the movie table with the 367 row key is no longer deleted.

1.  Verify the row from the movie table with the 231 row key is no longer a drama.

### Cleanup Large HDFS Files

1.  Before moving on, let’s remove the large backup files that were created in the previous steps.

1.  Delete `userbackup`.

1.  Delete `moviebackup`.

### Repairs

HBase can enter an inconsistent state. You will manually create an inconsistent state in HBase by closing a region and then repair it.

1.  Run the `hbck` and remember that this command will need to run as the hbase user.

1.  Check that final output of the hbck command says:

    ```console
    0 inconsistencies detected.
    Status: OK
    ```
    
    This shows that there are no errors in HBase.

1.  Looking at the output, find a line that looks similar to the output below and starts with movie:

    ```console
    2016-03-06 10:12:18,717 DEBUG [hbasefsck-pool1-t6] util.HBaseFsck: HRegionInfo read: {ENCODED => 2189031bb15d1d63845ee66b6c47b3b2, NAME => 'movie,,1425402913584.2189031bb15d1d63845ee66b6c47b3b2.',  STARTKEY => '', ENDKEY => ''}
    ```

1.  Once you have found the output line that starts with movie, copy the text from one single quote to the other as shown in bold below. The output in your terminal will not exactly match bolded line but will start with movie.

    ```console
    2016-03-06 10:12:18,717 DEBUG [hbasefsck-pool1-t6] util.HBaseFsck: HRegionInfo read: {ENCODED => 2189031bb15d1d63845ee66b6c47b3b2, NAME => 'movie,,1425402913584.2189031bb15d1d63845ee66b6c47b3b2.',STARTKEY => '', ENDKEY => ''}
    ```

1.  In the HBase shell, run the `close_region`:

    ```console
    hbase> close_region <outputfromhbck>`
    ```
    
    Given the output from step 4, the command would look like:

    ```console
    hbase> close_region 'movie,,1425402913584.2189031bb15d1d63845ee66b6c47b3b2.'
    ```
    The `close_region` command provides a way to manually close a region. After this command is issued, no RegionServer will be serving that region.

1.  In the HBase shell, run a scan on the movie

    This will report an error because in an earlier step you closed the region for the movie table:
    
    ```console
    ERROR: org.apache.hadoop.hbase.NotServingRegionException: Region  movie,,1425402913584.2189031bb15d1d63845ee66b6c47b3b2\. is not online on localhost,60020,1425653036092
    ```
    
    The movie table data cannot be accessed because there is no RegionServer serving that region.

1.  Run the `hbck` command again. Remember that this command will need to run as the hbase user.

1.  The final output of the hbck command will report an error:

    ```console
    2 inconsistencies detected.  
    Status: INCONSISTENT
    ```

1.  To fix this error, run the hbck program in the repair mode.

    This time, hbck will repair the issue, wait for a little bit and run the check again.

1.  Check that final output of the hbck command says:

    ```console
    0 inconsistencies detected.  
    Status: OK
    ```

    This shows that there are no errors in HBase - `hbck` has repaired them. The movie table will be accessible again.

1.  In the HBase shell, run a scan on the `movie` table to verify that it is accessible again.

### Result

You have now done backup and restore and additional administration duties.
