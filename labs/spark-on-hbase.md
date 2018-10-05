# Links for these files:

https://hortonworks.com/blog/spark-hbase-dataframe-based-hbase-connector/

https://hortonworks.com/blog/spark-hbase-connector-a-year-in-review/

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_spark-component-guide/content/spark-on-hbase.html

## Spark on HBase

## Objective:
Learn how to access and query HBase tables using Apache Spark.

### File locations:

### Successful outcome:
You will access and query HBase tables using Apache Spark.

### Lab Steps

Complete the setup guide

In this lab, we will see how to access and query HBase tables using Apache Spark. Spark can work on data present in multiple sources like a local filesystem, HDFS, Cassandra, Hbase, MongoDB etc. Now, we will see the steps for accessing HBase tables through spark.

----

1. Run spark-shell

		spark-shell --jars /usr/hdp/current/hbase-client/lib/hbase-client.jar,/usr/hdp/current/hbase-client/lib/hbase-server.jar,/usr/hdp/current/hbase-client/lib/hbase-shell.jar,/usr/hdp/current/hbase-client/lib/hbase-common.jar,/usr/hdp/current/hbase-client/lib/hbase-protocol.jar,/usr/hdp/current/hbase-client/lib/commons-collections-3.2.2.jar,/usr/hdp/current/hbase-client/lib/hbase-common.jar,/usr/hdp/current/hbase-client/lib/guava-12.0.1.jar

1. Now do following imports:

		import org.apache.hadoop.hbase.HBaseConfiguration
		import org.apache.hadoop.hbase.mapreduce.TableInputFormat
		import org.apache.hadoop.hbase.client.HBaseAdmin
		import org.apache.hadoop.hbase.{HTableDescriptor,HColumnDescriptor}
		import org.apache.hadoop.hbase.util.Bytes
		import org.apache.hadoop.hbase.client.{Put,HTable}


1. Now create hbase configuration object:

		val conf = HBaseConfiguration.create() 
		val tablename = "spark_Hbase"

1. Create Admin instance and set input format:

		conf.set(TableInputFormat.INPUT_TABLE,tablename)
		val admin = new HBaseAdmin(conf)

1. Create table:

		val tableDescription = new HTableDescriptor(tablename)
		tableDescription.addFamily(new HColumnDescriptor("cf".getBytes()));
		admin.createTable(tableDescription);

1. Check the create table exists or not:

		admin.isTableAvailable(tablename)

	If the table exists, it will return ‘True’ - get a list of tables:

		admin.getTableNames()

1. Now we will put some data into it:

		val table = new HTable(conf,tablename);
		for(x <- 1 to 10){
			var p = new Put(new String("row" + x).getBytes());
			p.add("colfamily1".getBytes(),"column1".getBytes(),new String("value" + x).getBytes());
			table.put(p);
		}

1. Flush your table mods:

		table.flushCommits()

1. Now we can create the HadoopRDD from the data present in HBase using newAPIHadoopRDD by InputFormat , output key and value class:

		val HBaseRDD = sc.newAPIHadoopRDD(config, classOf[TableInputFormat], classOf[org.apache.hadoop.hbase.io.ImmutableBytesWritable]. classOf[org.apache.hadoop.hbase.client.Result])

1. And we can perform all the transformations and actions on created RDD:

		val count = HBaseRdd.count()

### Result

So here you see that Spark can be used to Query tables in HBase.
