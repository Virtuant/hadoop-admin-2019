https://blogs.perficient.com/2015/09/09/some-ways-load-data-from-hdfs-to-hbase/

3 Ways to Load Data From HDFS to HBase
by Callen Wu on September 9th, 2015 | ~ 3 minute read

Apache HBase™ is the Hadoop database: a distributed, scalable, big data store. If you are importing into a new table, you can bypass the HBase API and write your content directly to the filesystem, formatted into HBase data files (HFiles). Your import will run much faster.

There are several ways to load data from HDFS to HBase. I practiced loading data from HDFS to HBase and listed my process step-by-step below.

Environment:
Pseudo-Distributed Local Install

Hadoop 2.6.0

HBase 0.98.13

1.Using ImportTsv to load txt to HBase
A) CREATE TABLE IN HBASE
command：create ‘tab3′,’cf’

3 Ways to Load Data From HDFS to HBase

B) UPLOADING SIMPLE1.TXT TO HDFS
command：bin/hadoop fs -copyFromLocal simple1.txt  /user/hadoop/simple1.txt

The context in the txt is:
1,tom
2,sam
3,jerry
4,marry
5,john

C) USING IMPORTTSV TO LOAD TXT TO HBASE
command：bin/hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=”,” -Dimporttsv.columns=HBASE_ROW_KEY,cf tab4 /user/hadoop/simple1.txt
ImportTsv execute result:

3 Ways to Load Data From HDFS to HBase

Data loaded into Hbase:

3 Ways to Load Data From HDFS to HBase

2.Using completebulkload to load txt to HBase

A) CREATING TABLE IN HBASE
command：create ‘hbase-tb1-003′,’cf’

3 Ways to Load Data From HDFS to HBase

B) USING IMPORTTSV TO GENERATE HFILE FOR TXT IN HDFS
command：bin/hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=”,” -Dimporttsv.bulk.output=hfile_tmp5 -Dimporttsv.columns=HBASE_ROW_KEY,cf hbase-tbl-003 /user/hadoop/simple1.txt

This command will be executed by MapReduce job:

3 Ways to Load Data From HDFS to HBase

As a result, the Hfile hfile_tmp5 is generated.

3 Ways to Load Data From HDFS to HBase

But the data wasn’t loaded into the Hbase table: hbase-tb1-003.

3 Ways to Load Data From HDFS to HBase

3.Using completebulkload to load Hfile to HBase

command: hadoop jar lib/hbase-server-0.98.13-hadoop2.jar completebulkload hfile_tmp5 hbase-tbl-003

3 Ways to Load Data From HDFS to HBase

Result:

3 Ways to Load Data From HDFS to HBase

Note: When we execute this command, Hadoop probably won’t be able to find the hbase dependency jar files through the exception of ClassNotFoundException. An easy way to resolve this is by adding HBase dependency jar files to the path ${HADOOP_HOME}/share/hadoop/common/lib. Do not forget htrace-core-*.**.jar.

This command is really an action of hdfs mv, and will not execute hadoop MapReduce. You can see the log; it is LoadIncrementalHFiles.

Apache HBase is an in-Hadoop database that delivers wide-column schema flexibility with strongly consistent reads and writes. Clients can access HBase data through either a native Java API, a Thrift or REST gateway, or now through a C API, making it very easy to access data. MapR-DB, yet another in-Hadoop database has the same HBase APIs, but provides enterprise-grade features for production deployments of HBase applications.
Put, Get and Scan are some of the prominent programming APIs that get used in the context of HBase applications. For certain write-heavy workloads, Put operations can get slow, so batching these Put operations is a commonly used technique to increase the overall throughput of the system. The following program illustrates a table load tool, which is a great utility program that can be used for batching Puts into an HBase/MapR-DB table. The program creates a simple HBase table with a single column within a column family and inserts 100000 rows with 100 bytes of data. The batch size for the Puts is set to 500 in this example.


