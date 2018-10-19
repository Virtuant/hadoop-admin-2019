## Spark on HBase

**Objective**: Learn how to access and query HBase tables using Apache Spark.

**Successful outcome**:  You will access and query HBase tables using Apache Spark.

In this lab, we will see how to access and query HBase tables using Apache Spark. Spark can work on data present in multiple sources like a local filesystem, HDFS, Cassandra, Hbase, MongoDB etc. Now, we will see the steps for accessing HBase tables through spark.

----

Run spark-shell

```
spark-shell --master yarn-client --driver-memory 512m --executor-memory 512m
```

Now do following imports:

```scala
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.mapreduce.TableInputFormat
import org.apache.hadoop.hbase.client.HBaseAdmin
import org.apache.hadoop.hbase.{HTableDescriptor,HColumnDescriptor}
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.client.{Put,HTable}
```

Now create hbase configuration object:

```scala
val conf = HBaseConfiguration.create() 
val tablename = "spark_Hbase"
```

Create Admin instance and set input format:

```scala
conf.set(TableInputFormat.INPUT_TABLE,tablename)
val admin = new HBaseAdmin(conf)
```

Create table:

```scala
val tableDescription = new HTableDescriptor(tablename)
tableDescription.addFamily(new HColumnDescriptor("cf".getBytes()));
admin.createTable(tableDescription);
```

Check the create table exists or not:

```scala
admin.isTableAvailable(tablename)
```

If the table exists, it will return ‘True’ - get a list of tables:

```scala
admin.getTableNames()
```

Now we will put some data into it:

```scala
val table = new HTable(conf,tablename);
for(x <- 1 to 10){
	var p = new Put(new String("row" + x).getBytes());
	p.add("colfamily1".getBytes(),"column1".getBytes(),new String("value" + x).getBytes());
	table.put(p);
}
```

Flush your table mods:

```scala
table.flushCommits()
```

Now we can create the HadoopRDD from the data present in HBase using newAPIHadoopRDD by InputFormat , output key and value class:

```scala
val HBaseRDD = sc.newAPIHadoopRDD(config, classOf[TableInputFormat], classOf[org.apache.hadoop.hbase.io.ImmutableBytesWritable]. classOf[org.apache.hadoop.hbase.client.Result])
```

And we can perform all the transformations and actions on created RDD:

```scala
val count = HBaseRdd.count()
```

### Result

So here you see that Spark can be used to Query tables in HBase.

{% comment %}
# Links for these files:

https://hortonworks.com/blog/spark-hbase-dataframe-based-hbase-connector/

https://hortonworks.com/blog/spark-hbase-connector-a-year-in-review/

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_spark-component-guide/content/spark-on-hbase.html

{% endcomment %}
