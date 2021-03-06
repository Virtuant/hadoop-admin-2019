## Using Salted Tables in HBase


Time Series Data

When dealing with stream processing of events, the most common use-case is time series data. This data might be generated by a sensor in a power grid, might represent stock ticks from a stock exchange, or it might represent the year a movie was released. Common in each case is that the data represents the event time.

Time data can pose a problem because of the way HBase arranges rows. Namely, rows are stored sorted in partitions defined for a distinct range. We refer to these partitions as regions. A region is assigned a range and stores row keys that belong to that range. The region range is defined by start key (inclusive) and stop key (exclusive).

The sequential, monotonically increasing nature of time series data causes all incoming data to be written to the same region. Since a single RegionServer hosts this region, all updates will be handled by a single machine, rather than spreading work across the cluster. This can cause regions to run ‘hot’, slowing down the perceived overall performance of the cluster. With monotonically increasing row keys, the speed of data insertion rests with a single machine.

In this exercise, you will run a program that adds the ratings for movies across all users that rated it. The program can create the four different row key types. You will run the exercise and observe the hotspotting characteristics of each row key type.

The program outputs write counters for the number of times a Put was performed on a region. In the first column, it shows the start key for the region. The second column shows the aggregate or total number of writes that region has handled. The final column shows the number of new writes since the last time the program output the statistics.

Each run of the program will recreate the table in HBase. Based on the row key type, a different pre-splitting of the HBase table is used. You can view the splits for each row key type by looking at the RowKeyType enum.

----

In Phoenix, drop and recreate the `iot_data' table.


```sql
0: jdbc:phoenix:localhost> drop table "iot_data";
No rows affected (1.644 seconds)
0: jdbc:phoenix:localhost> CREATE table "iot_data" ( 
	ROWKEY VARCHAR PRIMARY KEY, 
	"para"."deviceId" VARCHAR, 
	"para"."deviceParameter" VARCHAR, 
	"para"."deviceValue" varchar, 
	"para"."dateTime" date);
No rows affected (1.24 seconds)
```


<img width="1216" alt="screen shot 2018-09-01 at 5 58 11 pm" src="https://user-images.githubusercontent.com/558905/44950240-bbe72e80-ae10-11e8-87e7-151167370e2e.png">


```console
[centos@ip-10-0-0-34 data]$ hbase org.apache.hadoop.hbase.mapreduce.ImportTsv 
	-Dimporttsv.separator=, 
	-Dimporttsv.columns="HBASE_ROW_KEY,
		para:deviceParameter,
		para:deviceValue,
		para:deviceId,para:dateTime" 
	iot_data 
	hdfs://master1.hdp.com:/tmp/iot_data_cleaned.csv
```

### Create the Table with Salt Buckets

```sql
0: jdbc:phoenix:localhost> CREATE table "iot_data" ( 
	ROWKEY BIGINT PRIMARY KEY, 
	"para"."deviceId" VARCHAR, 
	"para"."deviceParameter" VARCHAR, 
	"para"."deviceValue" varchar, 
	"para"."dateTime" date) 
	salt_buckets=10;
No rows affected (1.24 seconds)
```

### Create a UDF

```java
import java.util.UUID;

public class GenerateUUID {
  
  public static final void main(String... args) {
    long count = Long.parseLong(args[0]);
    for (long i = 0L ; i < count; i++) {
	    UUID idOne = UUID.randomUUID();
	    log(idOne);
    }
  }
  
  private static void log(Object aObject){
    System.out.println( String.valueOf(aObject) );
  }
} 
```

### Or Create the Function (Ruby)

```ruby
hbase(main):070:0> def gen_splits(start_key,end_key,num_regions)
hbase(main):071:1>   results=[]
hbase(main):072:1>   range=end_key-start_key
hbase(main):073:1>   incr=(range/num_regions).floor
hbase(main):074:1>   for i in 1 .. num_regions-1
hbase(main):075:2>     results.push([i*incr+start_key].pack("N"))
hbase(main):076:2>   end
hbase(main):077:1>   return results
hbase(main):078:1> end
hbase(main):078:1> splits=gen_splits(100000,99999999,10)
```

```ruby
def gen_splits(start_key,end_key,num_regions)
	results=[]
	range=end_key-start_key
	incr=(range/num_regions).floor
	for i in 1 .. num_regions-1
	  results.push([i*incr+start_key].pack("N"))
	end
	return results
end
```


```sql                                                                                         
hbase(main):036:0> splits=gen_splits(100000,99999999,10)
```

```sql
hbase(main):036:0> create 'iot_data','para',SPLITS=>splits
```

### Results


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>
