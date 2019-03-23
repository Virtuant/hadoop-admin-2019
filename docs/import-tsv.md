## Ingesting Data with ImportTsv

**Objective**: You are going to bulk load data into an HBase table using the ImportTsv and CompleteBulkLoad tools. 

**Data file**: `~/data/hbase/data/hvac-tempdata.csv`

Perform a bulk import of comma-separated value data from local filesystem into HBase by uploading the raw data into HDFS; 
creating HFile-formatted files via the `ImportTsv` tool; and completing the bulk load process of submitting the 
HFile-formatted files to HBase using the `CompleteBulkLoad` tool.

----

### View the Test Data

We need to put the test data into our HDFS file system. View a few lines of the temperature data, 
making a note of the number of columns of comma-separated values:

    ```console
    head hvac-tempdata.csv 
    6/2/18,1:00:01,69,68,3,20,17
    6/3/18,2:00:01,70,73,17,20,18
    6/4/18,3:00:01,67,63,2,23,15
    6/5/18,4:00:01,68,74,16,9,3
    6/6/18,5:00:01,67,56,13,28,4
    6/7/18,6:00:01,70,58,12,24,2
    6/8/18,7:00:01,70,73,20,26,16
    6/9/18,8:00:01,66,69,16,9,9
    6/10/18,9:00:01,65,57,6,5,12
    ```

A possible row-key would be a combination of the date and time fields (the first two comma-separated values). 
Run this command to combine those two fields into a single field:

    ```console
    sed 's/,/-/' < hvac-tempdata.csv > tempdata.csv

    head tempdata.csv 
    6/1/18-0:00:01,66,58,13,20,4
    6/2/18-1:00:01,69,68,3,20,17
    6/3/18-2:00:01,70,73,17,20,18
    6/4/18-3:00:01,67,63,2,23,15
    6/5/18-4:00:01,68,74,16,9,3
    6/6/18-5:00:01,67,56,13,28,4
    6/7/18-6:00:01,70,58,12,24,2
    6/8/18-7:00:01,70,73,20,26,16
    6/9/18-8:00:01,66,69,16,9,9
    6/10/18-9:00:01,65,57,6,5,12
    ```
 
Observe the new row format is `<date>-<time>` followed by 5 values.

### Upload the test data file to HDFS:

    ```console
    hdfs dfs -mkdir hvac
    hdfs dfs -put tempdata.csv hvac
    ```

The bulk load tasks will be run as the hbase user. Change the permissions of our directory and data to allow the hbase user access to the directory and its contents:

    ```console
    hdfs dfs –chmod -R 777 hvac
    ```
    
### Run the ImportTsv Tool on the `tempdata.csv` file:

    ```console
    hbase org.apache.hadoop.hbase.mapreduce.ImportTsv \ 
    -Dimporttsv.separator=',' \
    -Dimporttsv.bulk.output=/user/student/hvac/tmp \
    -Dimporttsv.columns=HBASE_ROW_KEY,cf1:c1,cf1:c2,cf1:c3,cf1:c4,cf1:c5 \ 
    hvac /user/student/hvac/tempdata.csv
    ```
    
LOTS of output truncated here …

	```
    15/04/17 11:56:52 INFO mapreduce.Job: Running job: job_1429023797784_0010 
    15/04/17 11:56:58 INFO mapreduce.Job: Job job_1429023797784_0010 running in uber mode : false
    15/04/17 11:56:58 INFO mapreduce.Job:	map 0% reduce 0%
    15/04/17 11:57:13 INFO mapreduce.Job:	map 100% reduce 0%
    15/04/17 11:57:22 INFO mapreduce.Job:	map 100% reduce 100%
    15/04/17 11:57:23 INFO mapreduce.Job: Job job_1429023797784_0010 completed successfully
    15/04/17 11:57:23 INFO mapreduce.Job: Counters: 50 File System Counters
    FILE: Number of bytes read=829605 FILE: Number of bytes written=1943633
    ```

### Confirm the Output of the ImportTsv tool:

    ```console
    hdfs dfs -ls -R hvac/tmp 
    Found 2 items
    -rw-r--r--	3 root root	0 2016-04-17 11:57 hvac/tmp/_SUCCESS
    drwxr-xr-x	- root root	0 2016-04-17 11:57 hvac/tmp/cf1
    -rw-r--r--	3 root hdfs	7511 2016-05-15 06:54 hvac/tmp/cf1/9b7e6430296144ed808bd0faee1daef9
    ```
 
We need to make the permissions of these new files open for the hbase user:

    ```console
    hdfs dfs -chmod -R 777 hvac
    ```
    
### Run the `completebulkload` tool

    ```console
    hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /user/student/hvac/tmp hvac
 
    … <output truncated> …

    13:44:57 INFO mapreduce.LoadIncrementalHFiles: Trying to load hfile=hdfs://node1:8020/user/student/hvac/tmp/cf1/263694b200834949948b34df676eb76b first=6/1/16@0:00:01 last=6/9/18@9:45:56
    ```

9. Confirm that the bulk load completed successfully by inspecting the hvac table from the HBase shell:

    ```console
    hbase(main):001:0> count 'hvac'
    Current count: 1000, row: 6/2/16@9:45:56
    Current count: 2000, row: 6/30/16@9:45:56 
    2500 row(s) in 0.8400 seconds	=> 2500
	```

Now scan the table:

    ```console
	hbase(main):002:0> scan 'hvac'
    6/9/16@9:45:56	column=cf1:c2, timestamp=1429292640140, value=72 
    6/9/16@9:45:56	column=cf1:c3, timestamp=1429292640140, value=9 
    6/9/16@9:45:56	column=cf1:c4, timestamp=1429292640140, value=30 
    6/9/16@9:45:56	column=cf1:c5, timestamp=1429292640140, value=5 
    2500 row(s) in 6.7060 seconds
    ```

### Results

You performed a bulk import of comma-separated value data from local filesystem into HBase by uploading 
the raw data into HDFS; creating HFile-formatted files via the ImportTsv tool; and completing the bulk 
load process of submitting the HFile-formatted files to HBase using the CompleteBulkLoad tool.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>
