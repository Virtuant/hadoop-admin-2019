# Here are some links for this lab:

https://community.hortonworks.com/articles/4942/import-csv-data-into-hbase-using-importtsv.html

https://community.hortonworks.com/questions/66929/reducer-is-running-extremely-slow-in-bulk-import-t.html

https://community.hortonworks.com/questions/90455/join-two-tables-into-one-hbase-table.html

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_hbase-java-api-reference/org/apache/hadoop/hbase/mapreduce/package-summary.html

## Lab: Ingesting Data with ImportTsv
## Objective:
To Bulk Load data into an HBase table using the ImportTsv and CompleteBulkLoad tools.

## File locations:
'/root/hbase-labs/hvac-tempdata.csv' 

## Successful outcome:
You will:
Perform a bulk import of comma-separated value data from local filesystem into HBase by uploading the raw data into HDFS; creating HFile-formatted files via the ImportTsv tool; and completing the bulk load process of submitting the HFile-formatted files to HBase using the CompleteBulkLoad tool.

## Before you begin 
Successfully complete the Installation Lab

## Related Lesson:
Ingesting Data into HBase
Lab Steps
Perform the following steps:





| `git status` | List all *new or modified* files |
| `git diff` | Show file differences that **haven't been** staged |

You are going to bulk load data into an HBase table using the ImportTsv and CompleteBulkLoad tools. Perform a bulk import of comma-separated value data from local filesystem into HBase by uploading the raw data into HDFS; creating HFile-formatted files via the ImportTsv tool; and completing the bulk load process of submitting the HFile-formatted files to HBase using the CompleteBulkLoad tool.

**Files**: [hvac-tempdata.csv](https://www.dropbox.com/sh/bny8lvxw5gqzytl/AADNBpFCI5AiBQZ3flKB5Shna?dl=0)

> Note: if you can't get the file to your system, follow instructs at the bottom of this page.

----

1. We need to put the test data into our HDFS file system. View a few lines of the temperature data, making a note of the number of columns of comma-separated values:

    ```console
    # head hvac-tempdata.csv 
    6/2/13,1:00:01,69,68,3,20,17
    6/3/13,2:00:01,70,73,17,20,18
    6/4/13,3:00:01,67,63,2,23,15
    6/5/13,4:00:01,68,74,16,9,3
    6/6/13,5:00:01,67,56,13,28,4
    6/7/13,6:00:01,70,58,12,24,2
    6/8/13,7:00:01,70,73,20,26,16
    6/9/13,8:00:01,66,69,16,9,9
    6/10/13,9:00:01,65,57,6,5,12
    ```

2.	A possible row-key would be a combination of the date and time fields (the first two comma-separated values). Run this command to combine those two fields into a single field:

    ```console
    # sed 's/,/-/' < hvac-tempdata.csv > tempdata.csv

    # head tempdata.csv 
    6/1/13-0:00:01,66,58,13,20,4
    6/2/13-1:00:01,69,68,3,20,17
    6/3/13-2:00:01,70,73,17,20,18
    6/4/13-3:00:01,67,63,2,23,15
    6/5/13-4:00:01,68,74,16,9,3
    6/6/13-5:00:01,67,56,13,28,4
    6/7/13-6:00:01,70,58,12,24,2
    6/8/13-7:00:01,70,73,20,26,16
    6/9/13-8:00:01,66,69,16,9,9
    6/10/13-9:00:01,65,57,6,5,12
    ```
 
    Observe the new row format is `<date>-<time>` followed by 5 values.
    
3.	Upload the test data file to HDFS:

    ```console
    # hdfs dfs -mkdir hvac
    # hdfs dfs -put tempdata.csv hvac
    ```

4. The bulk load tasks will be run as the hbase user. Change the permissions of our directory and data to allow the hbase user access to the directory and its contents:

    ```console
    # hdfs dfs –chmod -R 777 hvac
    ```
    
5. Run the ImportTsv tool on the `tempdata.csv` file:

    ```console
    # hbase org.apache.hadoop.hbase.mapreduce.ImportTsv \ 
    -Dimporttsv.separator=',' \
    -Dimporttsv.bulk.output=/user/student/hvac/tmp \
    -Dimporttsv.columns=HBASE_ROW_KEY,cf1:c1,cf1:c2,cf1:c3,cf1:c4,cf1:c5 \ 
    hvac /user/student/hvac/tempdata.csv
    
    …<LOTS of output truncated here>…

    15/04/17 11:56:52 INFO mapreduce.Job: Running job: job_1429023797784_0010 
    15/04/17 11:56:58 INFO mapreduce.Job: Job job_1429023797784_0010 running in uber mode : false
    15/04/17 11:56:58 INFO mapreduce.Job:	map 0% reduce 0%
    15/04/17 11:57:13 INFO mapreduce.Job:	map 100% reduce 0%
    15/04/17 11:57:22 INFO mapreduce.Job:	map 100% reduce 100%
    15/04/17 11:57:23 INFO mapreduce.Job: Job job_1429023797784_0010 completed successfully
    15/04/17 11:57:23 INFO mapreduce.Job: Counters: 50 File System Counters
    FILE: Number of bytes read=829605 FILE: Number of bytes written=1943633

    … <output truncated> …
    ```

6. Confirm the output of the ImportTsv tool:

    ```console
    # hdfs dfs -ls -R hvac/tmp 
    Found 2 items
    -rw-r--r--	3 root root	0 2016-04-17 11:57 hvac/tmp/_SUCCESS
    drwxr-xr-x	- root root	0 2016-04-17 11:57 hvac/tmp/cf1
    -rw-r--r--	3 root hdfs	7511 2016-05-15 06:54 hvac/tmp/cf1/9b7e6430296144ed808bd0faee1daef9
    ```
 
7. We need to make the permissions of these new files open for the hbase user:

    ```console
    # hdfs dfs -chmod -R 777 hvac
    ```
    
8. Run the `completebulkload` tool

    ```console
    # hbase org.apache.hadoop.hbase.mapreduce.LoadIncrementalHFiles /user/student/hvac/tmp hvac
 
    … <output truncated> …

    13:44:57 INFO mapreduce.LoadIncrementalHFiles: Trying to load hfile=hdfs://node1:8020/user/student/hvac/tmp/cf1/263694b200834949948b34df676eb76b first=6/1/16@0:00:01 last=6/9/13@9:45:56
    ```

9. Confirm that the bulk load completed successfully by inspecting the hvac table from the HBase shell:

    ```console
    # hbase shell
    hbase(main):001:0> count 'hvac'
    Current count: 1000, row: 6/2/16@9:45:56
    Current count: 2000, row: 6/30/16@9:45:56 
    2500 row(s) in 0.8400 seconds	=> 2500

    hbase(main):002:0> scan 'hvac'

    … <output truncated> …

    6/9/16@9:45:56	column=cf1:c2, timestamp=1429292640140, value=72 
    6/9/16@9:45:56	column=cf1:c3, timestamp=1429292640140, value=9 
    6/9/16@9:45:56	column=cf1:c4, timestamp=1429292640140, value=30 
    6/9/16@9:45:56	column=cf1:c5, timestamp=1429292640140, value=5 
    2500 row(s) in 6.7060 seconds
    ```

### Result

You performed a bulk import of comma-separated value data from local filesystem into HBase by uploading the raw data into HDFS; creating HFile-formatted files via the ImportTsv tool; and completing the bulk load process of submitting the HFile-formatted files to HBase using the CompleteBulkLoad tool.

### File Instructions from Dropbox

If your scenario doesn't allow the file download to the system, we have provided a Dropbox just for this purpose.

Go to root on the host (you may simply have to exit):

	[root@sandbox-host ~]#

Change user to student:

	[root@sandbox-host ~]# su - student
	Last login: Wed Nov  1 14:31:53 UTC 2017 on pts/6
	[student@sandbox-host ~]$

Start Dropbox:

	[student@sandbox-host ~]$ dropbox.py start


And go to this directory:

	[student@sandbox-host ~]$ cd Dropbox/Student/

Wait several seconds, and (your files may vary):

	[student@sandbox-host Student]$ ll
	total 248
	-rw-rw-r-- 1 student student 240531 Oct 31 19:16 hvac-tempdata.csv
	[student@sandbox-host Student]$

Now copy to your comtainer:

	[student@sandbox-host Student]$ sudo docker cp *.csv d27:/home/student/hvac-tempdata.csv

Now you can go back to the container.

