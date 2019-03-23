## Importing Data into HBase from IoT Tables

**Objective**: In this exercise you will import data from MySQL into HBase.

**Exercise directory**: `~/data/hbase/data`

**HBase tables**:       `iotdata`  

In this exercise you will import data from MongoDB into HBase. Your company was using MongoDB and started hitting scale issues as the traffic grew. You have been tasked with migrating the data from MongoDB to HBase.

----

### Load data Into MongoDB

One of the first cases we get to see with Hbase is loading it up with Data, most of the time we will have some sort of data in some format like CSV availalble and we would like to load it in Hbase, lets take a quick look on how does the procedure looks like. Lets examine our example data by looking at the simple structure that we have for an industrial sensor:

|Device ID|Device Parameter|Device Value|Device ID|Date Time|
| ---- | ---- | ---- | ---- | ---- |
|ObjectId(5a81b5395882b86112555f70)	|Temperature|	27	|SBS05	|2018-02-12 15:39:37.050 UTC|
|ObjectId(5a81b5395882b86112555f71)	|Humidity	|59	|SBS05	|2018-02-12 15:39:37.801 UTC|
|ObjectId(5a81b53a5882b86112555f72)|	Sound	|130	|SBS04	|2018-02-12 15:39:38.629 UTC|
|ObjectId(5a81b53b5882b86112555f73)|	Humidity|	75	|SBS05	|2018-02-12 15:39:39.272 UTC|
|ObjectId(5a81b53b5882b86112555f74)|	Temperature	|33	|SBS02|	2018-02-12 15:39:39.613 UTC|
|ObjectId(5a81b53c5882b86112555f75)|	Sound	|102|	SBS03|	2018-02-12 15:39:40.363 UTC|
|ObjectId(5a81b53c5882b86112555f76)	|Temperature|	18	|SBS02	|2018-02-12 15:39:40.663 UTC|
|ObjectId(5a81b53c5882b86112555f77)|	Flow	|64	|SBS05|	2018-02-12 15:39:40.678 UTC|
|ObjectId(5a81b53d5882b86112555f78)|	Temperature|	28	|SBS04	|2018-02-12 15:39:41.141 UTC|
| ... |

View the file in Linux (more command).

```mongo
[centos@ip-10-0-0-237 data]$ more iot_data.csv
_id,deviceParameter,deviceValue,deviceId,dateTime
ObjectId(5a81b5395882b86112555f70),Temperature,27,SBS05,2018-02-12 15:39:37.050 UTC
ObjectId(5a81b5395882b86112555f71),Humidity,59,SBS05,2018-02-12 15:39:37.801 UTC
ObjectId(5a81b53a5882b86112555f72),Sound,130,SBS04,2018-02-12 15:39:38.629 UTC
ObjectId(5a81b53b5882b86112555f73),Humidity,75,SBS05,2018-02-12 15:39:39.272 UTC
ObjectId(5a81b53b5882b86112555f74),Temperature,33,SBS02,2018-02-12 15:39:39.613 UTC
ObjectId(5a81b53c5882b86112555f75),Sound,102,SBS03,2018-02-12 15:39:40.363 UTC
ObjectId(5a81b53c5882b86112555f76),Temperature,18,SBS02,2018-02-12 15:39:40.663 UTC
ObjectId(5a81b53c5882b86112555f77),Flow,64,SBS05,2018-02-12 15:39:40.678 UTC
ObjectId(5a81b53d5882b86112555f78),Temperature,28,SBS04,2018-02-12 15:39:41.141 UTC
ObjectId(5a81b53d5882b86112555f79),Humidity,69,SBS03,2018-02-12 15:39:41.804 UTC
ObjectId(5a81b53e5882b86112555f7a),Temperature,19,SBS04,2018-02-12 15:39:42.350 UTC
ObjectId(5a81b53e5882b86112555f7b),Temperature,28,SBS05,2018-02-12 15:39:42.593 UTC
ObjectId(5a81b53f5882b86112555f7c),Temperature,31,SBS04,2018-02-12 15:39:43.070 UTC
ObjectId(5a81b53f5882b86112555f7d),Sound,133,SBS05,2018-02-12 15:39:43.961 UTC
ObjectId(5a81b5405882b86112555f7e),Flow,99,SBS02,2018-02-12 15:39:44.031 UTC
ObjectId(5a81b5405882b86112555f7f),Humidity,65,SBS04,2018-02-12 15:39:44.667 UTC
ObjectId(5a81b5415882b86112555f80),Humidity,90,SBS05,2018-02-12 15:39:45.260 UTC
ObjectId(5a81b5415882b86112555f81),Flow,89,SBS05,2018-02-12 15:39:45.460 UTC
ObjectId(5a81b5425882b86112555f82),Sound,140,SBS03,2018-02-12 15:39:46.389 UTC
ObjectId(5a81b5425882b86112555f83),Flow,61,SBS02,2018-02-12 15:39:46.569 UTC
ObjectId(5a81b5435882b86112555f84),Temperature,20,SBS05,2018-02-12 15:39:47.087 UTC
ObjectId(5a81b5435882b86112555f85),Temperature,16,SBS05,2018-02-12 15:39:47.635 UTC
ObjectId(5a81b5435882b86112555f86),Flow,64,SBS01,2018-02-12 15:39:47.682 UTC
ObjectId(5a81b5435882b86112555f87),Temperature,21,SBS04,2018-02-12 15:39:47.942 UTC
ObjectId(5a81b5445882b86112555f88),Flow,97,SBS01,2018-02-12 15:39:48.046 UTC
ObjectId(5a81b5455882b86112555f89),Sound,133,SBS03,2018-02-12 15:39:49.058 UTC
ObjectId(5a81b5455882b86112555f8a),Sound,100,SBS05,2018-02-12 15:39:49.870 UTC
ObjectId(5a81b5465882b86112555f8b),Temperature,15,SBS02,2018-02-12 15:39:50.139 UTC
ObjectId(5a81b5465882b86112555f8c),Temperature,20,SBS03,2018-02-12 15:39:50.406 UTC
ObjectId(5a81b5465882b86112555f8d),Temperature,21,SBS03,2018-02-12 15:39:50.855 UTC
ObjectId(5a81b5475882b86112555f8e),Humidity,74,SBS02,2018-02-12 15:39:51.524 UTC
ObjectId(5a81b5485882b86112555f8f),Humidity,57,SBS02,2018-02-12 15:39:52.111 UTC
ObjectId(5a81b5485882b86112555f90),Sound,113,SBS02,2018-02-12 15:39:52.909 UTC
ObjectId(5a81b5495882b86112555f91),Flow,99,SBS05,2018-02-12 15:39:53.127 UTC
```

Now let's load the data into MongoDB. Go to the exercise directory and make sure you can see the file. 

So now let's check MongoDB:

```mongo
[centos@ip-10-0-0-237 data]$ mongo
MongoDB shell version v4.0.1
connecting to: mongodb://127.0.0.1:27017
MongoDB server version: 4.0.1
```

And let's see if our data base has been created:

```mongo
> show databases;
admin    0.000GB
config   0.000GB
local    0.000GB
ourcity  0.022GB
```

It has, go into it and delete it:

```mongo
        > use ourcity
        switched to db ourcity
        > db.dropDatabase()
        { "dropped" : "ourcity", "ok" : 1 }
        > exit
        bye
        [centos@ip-10-0-0-54 data]$ 
```

Then run:

```mongo
[centos@ip-10-0-0-237 data]$ mongoimport iot_data.csv --type csv --headerline -d ourcity -c iotdata
2018-08-11T19:45:58.435+0000    connected to: localhost
2018-08-11T19:46:01.434+0000    [##############..........] ourcity.iotdata      19.4MB/32.8MB (59.1%)
2018-08-11T19:46:03.536+0000    [########################] ourcity.iotdata      32.8MB/32.8MB (100.0%)
2018-08-11T19:46:03.536+0000    imported 426878 documents
```

We have imported 42,678 documents to the MongoDB data base `iotdata`.

Now go back into Mongo and look at the data in the db:

```mongo
> use ourcity
switched to db ourcity
```

Let's check our data:

```mongo
> db.iotdata.findOne()
{
        "_id" : "ObjectId(5a81b53b5882b86112555f73)",
        "deviceParameter" : "Humidity",
        "deviceValue" : 75,
        "deviceId" : "SBS05",
        "dateTime" : "2018-02-12 15:39:39.272 UTC"
}
```

sure enough, it's there!


### Creating the HBase Table

Go into the Hbase Shell, and create the example table:

```hbase
hbase(main):001:0> create 'sensor', 'id', 'parameter', 'value', 'deviceid', 'datetime'
```

Now lets make sure the table was created and examine the structure:

```hbase
hbase(main):001:0> list
```

And describe the table:

```hbase
hbase(main):002:0> describe 'sensor'
Table sensor is ENABLED
sensor
COLUMN FAMILIES DESCRIPTION
{NAME => 'datetime', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => '
false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', I
N_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
{NAME => 'deviceid', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => '
false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', I
N_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
{NAME => 'id', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => 'false'
, DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_MEMO
RY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
{NAME => 'parameter', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE =>
'false', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false',
IN_MEMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
{NAME => 'value', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_CELLS => 'FALSE', CACHE_DATA_ON_WRITE => 'fal
se', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', MIN_VERSIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'ROW', CACHE_INDEX_ON_WRITE => 'false', IN_M
EMORY => 'false', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE => 'true', BLOCKSIZE => '65536'}
5 row(s)
Took 0.5779 seconds
```

https://oss.sonatype.org/content/repositories/releases/org/mongodb/mongodb-driver/3.8.0/mongodb-driver-3.8.0.jar


Now, exit the shell by typing 'exit' and lets load some data!

### Loading the Data

```console
    sqoop import \
          --connect jdbc:mongodb://localhost:3306/db_bdp \
          --driver com.mysql.jdbc.Driver \
          --username root \
          --table employee \
          --hbase-create-table \
          --hbase-table employee_details \
          --column-family employees \
          --hbase-row-key id -m 1
```

lets put the hbase.csv file in HDFS, you may SCP it first to the cluster by using the following command
macbook-ned> scp hbase.csv root@sandbox.hortonworks.com:/home/hbase
now put in HDFS using the following command
hbase> hadoop dfs -copyFromLocal hbase.csv /tmp
we shall now execute the Loadtsv statement as following
hbase> hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=,  -Dimporttsv.columns="HBASE_ROW_KEY,id,temp:in,temp:out,vibration,pressure:in,pressure:out" sensor hdfs://sandbox.hortonworks.com:/tmp/hbase.csv
once the mapreduce job is completed, return back to hbase shell and execute
hbase(main):001:0> scan sensor
you should now see the data in the table
Remarks

Importtsv statement generates massive amount of logs, so make sure you have enough space in /var/logs, its always better to have it mounted on a seperate directories in real cluster to avoid operational stop becuase of logs filling the partition.

### Summary


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>