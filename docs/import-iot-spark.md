https://medium.com/hashmapinc/3-steps-for-bulk-loading-1m-records-in-20-seconds-into-apache-phoenix-99b77ad87387

Steps for Bulk Loading 1M Records in 20 Seconds Into Apache Phoenix
Using Apache Spark for High Performance Data Loading into Apache Phoenix/HBase

by Pavan Guthikonda and Preetpal Singh

As consultants, engineers, and developers in the Big Data space working with a variety of platforms and toolsets, one of the common technical use cases we encounter with clients is the need to bulk load data with extremely high performance into Apache Phoenix/HBase, ultimately performing SQL queries on the data via Phoenix.
A High Performance Approach
While we have been down the path of other options such as following a standard MemStore and WAL-based write pipeline, we’ve found it’s very difficult to get an order of magnitude increase in data ingestion speed while also optimizing CPU and memory utilization for a HBase cluster.
An approach that we have seen work well to address the above use case is to construct Phoenix compatible HFiles directly and upload them to HBase using Apache Spark. Data loading through HFiles is efficient as you are skipping the use of resources such as server memory (JVM heap in general and Memstore in particular), write-ahead log (WAL), compactions, flushes, and garbage collection.
We have achieved data loads of 1 million records in 20 seconds or less using this approach.
Please keep in mind that the referenced throughput is a function of the size and overall system resources available in your cluster — your mileage may vary. Our benchmark tests were done on a 9 node HDP 2.6.3 cluster with 8 cores and 32GB RAM HBase region servers.
So Where Do I Start?
If you are trying to achieve high throughput for your data loads, you can find a number of articles describing how to load data to HBase using Spark, but often these posts do not provide guidance on a number of important aspects of the process and can lead to failure.
As an example, for a particular bulk load dataset…
What is the total number of HFiles generated based on the number of Spark partitions for the dataset that is being bulk loaded?
How should sequencing/ordering of row keys be handled?
What about byte serialization for compatibility to Phoenix data types?
Being able to write HFiles is a starting point, but it takes a bit more to get your solution working and into production.
3 Steps for High Performance
Below, we’ve provided the 3 key steps to achieve high performance data loading into Phoenix/HBase with Spark. Note that this post is more focused on Phoenix bulk loads rather than HBase bulk loads. We find that more has been written on the HBase specific topic than Phoenix Bulk load.
Sample code below illustrates the steps to be taken to write a Spark based bulk loading job for Phoenix.
Step 1: Prepare Spark RDD from the Data File
Below is sample DDL for the Phoenix table:
CREATE TABLE HTOOL_P (
U_ID BIGINT NOT NULL,
TIME_IN_ISO VARCHAR,
VAL VARCHAR,
BATCH_ID VARCHAR,
JOB_ID VARCHAR,
POS_ID VARCHAR,
CONSTRAINT pk PRIMARY KEY (U_ID)) COMPRESSION=’SNAPPY’;
Load data from a CSV file:


We need to transform the DataFrame to an RDD of format ((ImmutableBytesWritable (rowkey)), KeyValue)) pairs so that we can write HFiles using saveAsNewAPIHadoopFile method available in Spark.
Explained below is how we transform our DataFrame into our required RDD format.

In the above code, DataFrame is sorted by column names and we are generating an RDD of (rowkey, (column family, column name, column value)) tuples using flatmap on the sorted DataFrame.
Important Tip
Make sure your RDD is sorted by row key, column family and column name in the given order and partitioned equal to the number of regions of the HBase table for which you are ingesting data. This is very important so that you don’t run into issues while loading the HFile to HBase.

Here we are transforming our RDD into the final required format of (immutable rowkey, KeyValue) pairs. Everything in HBase is stored in byte array format and therefore we are converting rowkey, column family, column name, and values to the byte array format.
Step 2: Generate HFiles (Serialized Phoenix formats included) from Spark RDDs
The below code snippet uses HBase to configure a Spark job that performs an incremental load into the given table:
HFileOutputFormat2.configureIncrementalLoad(org.apache.hadoop.mapreduce.Job job, HTableDescriptor tableDescriptor, RegionLocator regionLocator)
Check out the link here for more information on the method.
After setting this up, you can call Spark’s “saveAsNewAPIHadoopFile” method of the PairRDD to write HFiles to the given location.

  
Step 3: Upload HFiles to HDFS/HBASE.
Lastly, the below code snippet will load HFiles to HDFS.

If you’ve followed the above steps, then you’ve now taken a high performance approach to bulk loading data into Phoenix/HBase with Apache Spark.
In summary, the 3 steps required for this approach are:
Prepare RDD from the Data File
Generate HFiles (Serialized Phoenix formats included) from Spark RDDs
Upload HFiles to HDFS/HBase
We hope this post has helped you in thinking through an approach to leveraging Spark and Phoenix for high performance data loading and you too will soon be able to achieve load times of 1 million records in less than 20 seconds for your applications.
If you need consulting or development assistance in this area or just need general help with HBase, Phoenix, Spark, and other open source Big Data technologies, please reach out to us.
