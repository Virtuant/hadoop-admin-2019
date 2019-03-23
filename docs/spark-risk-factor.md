## Apache Hive – Data ETL

**Objective**: In this lab we will introduce Apache Spark. 

**Exercise directory**: `~/data/iot`

In an earlier lab you have learned how to load data into HDFS and then manipulate it using Hive. We are using the Truck sensor data to better understand risk associated with every driver. This section will teach you how to compute risk using Apache Spark.

MapReduce has been useful, but the amount of time it takes for the jobs to run can at times be exhaustive. Furthermore, MapReduce jobs only work for a specific set of use cases. There is a need for computing framework that works for a wider set of use cases.

Apache Spark was designed to be a fast, general-purpose, easy-to-use computing platform. It extends the MapReduce model and takes it to a whole other level. The speed comes from the in-memory computations. Applications running in memory allow for much faster processing and response.

Spark is a fast, in-memory data processing engine with elegant and expressive development APIs in Scala, Java, Python and R that allow data workers to efficiently execute machine learning algorithms that require fast iterative access to datasets. Spark on Hadoop YARN enables deep integration with Hadoop and other YARN enabled workloads in the enterprise.

You can run batch application such as MapReduce types jobs or iterative algorithms that build upon each other. You can also run interactive queries and process streaming data with your application. Spark also provides a number of libraries which you can easily use to expand beyond the basic Spark capabilities such as Machine Learning algorithms, SQL, streaming, and graph processing. Spark runs on Hadoop clusters such as Hadoop YARN or Apache Mesos, or even in a Standalone Mode with its own scheduler. 

Let’s get started!

----

### Configure Spark Services using Ambari

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Log on to Ambari Dashboard</h4>

At the bottom left corner of the services column, check that Spark2 and Zeppelin Notebook are running.

![note](https://user-images.githubusercontent.com/558905/40528492-37597500-5fbf-11e8-96a1-f4d206df64ab.png) If these services are disabled, start these services.

![ambari-dash-running-spark-800x651](https://user-images.githubusercontent.com/558905/54868594-9433c580-4d64-11e9-9bc4-8bc3e6cfe484.jpg)

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Open Zeppelin interface </h4>

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) Probably using URL: http://master1.hadoop.com.com:9995/

You should see a Zeppelin Welcome Page:

![welcome-to-zeppelin-800x457](https://user-images.githubusercontent.com/558905/54868616-95fd8900-4d64-11e9-9479-4f8aadb35b08.jpg)

Optionally, if you want to find out how to access the Spark shell to run code on Spark refer to Appendix A.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Create a Zeppelin Notebook</h4>

Click on a Notebook tab at the top left and select Create new note. Name your notebook: `Compute Riskfactor with Spark`

![create-new-notebook-800x460](https://user-images.githubusercontent.com/558905/54868599-9433c580-4d64-11e9-8a18-99384ec1dcc7.jpg)

![new-spark-note-800x457](https://user-images.githubusercontent.com/558905/54868609-9564f280-4d64-11e9-8035-9866609ca18a.jpg)

### Create a Hive Context

For improved Hive integration, ORC file support has been added for Spark. This allows Spark to read data stored in ORC files. Spark can leverage ORC file’s more efficient columnar storage and predicate pushdown capability for even faster in-memory processing. HiveContext is an instance of the Spark SQL execution engine that integrates with data stored in Hive. The more basic SQLContext provides a subset of the Spark SQL support that does not depend on Hive. It reads the configuration for Hive from hive-site.xml on the classpath.

Import sql libraries:

If you already have a riskfactor table on your system you must remove it so that you can populate it again using Spark. Copy and paste the following code into your Zeppelin notebook, then click the play button. Alternatively, press shift+enter to run the code.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Instantiate SparkSession</h4>

```spark
%spark2
val hiveContext = new org.apache.spark.sql.SparkSession.Builder().getOrCreate()
```

![instantiate_hivecontext_hello_hdp_lab4](https://user-images.githubusercontent.com/558905/54868605-94cc5c00-4d64-11e9-9082-626a31282718.png)

### Create a RDD from Hive Context

All operations fall into one of two types: transformations or actions.

* Transformation operations, as the name suggests, create new datasets from an existing RDD and build out the processing DAG that can then be applied on the partitioned dataset across the YARN cluster. Transformations do not return a value. In fact, nothing is evaluated during the definition of these transformation statements. 
* Spark just creates these Direct Acyclic Graphs or DAG, which will only be evaluated at runtime. We call this lazy evaluation.
* An Action operation, on the other hand, executes a DAG and returns a value.

#### Read CSV Files into Apache Spark

In this lab we use the CSV files we stored in HDFS in previous labs. Additionally, we will leverage Global Temporary Views on SparkSessions to programmatically query DataFrames using SQL.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Import CSV data into a data frame without a user defined schema

```spark
%spark2
/**
 * Let us first see what temporary views are already existent on our Sandbox
 */
hiveContext.sql("SHOW TABLES").show()
```

If you have not created any temporary views in this Spark instance there should not be any tables:

![empty-set](https://user-images.githubusercontent.com/558905/54868602-94cc5c00-4d64-11e9-894a-b069f2b8a1b4.jpg)

First we must read data from HDFS, in this case we are reading from a csv file without having defined the schema first:

```spark
%spark2
val geoLocationDataFrame = spark.read.format("csv").option("header", "true").load("hdfs:///tmp/data/geolocation.csv")

/**
 * Now that we have the data loaded into a DataFrame, we can register a temporary view.
 */
geoLocationDataFrame.createOrReplaceTempView("geolocation")
```

Let’s verify that the data in our CSV file was properly loaded into our data frame:

```spark
%spark2
hiveContext.sql("SELECT * FROM geolocation LIMIT 15").show()
```

![select-geo-from-csv-800x302](https://user-images.githubusercontent.com/558905/54868612-9564f280-4d64-11e9-9d7d-8ccddc731c48.jpg)

Note that our data is casted onto the appropriate type when we register it as a temporary view:

```spark
%spark2
hiveContext.sql("DESCRIBE geolocation").show()
```

![describe-geo-800x438](https://user-images.githubusercontent.com/558905/54868601-94cc5c00-4d64-11e9-98d3-44917708affd.jpg)

Alternatively, we can define our schema with specific types, we will explore this option on the next paragraph.

Import CSV data into a data frame with a user defined schema:

```spark
%spark2
/**
 * The SQL Types library allows us to define the data types of our schema
 */
import org.apache.spark.sql.types._

/**
 * Recall from the previous tutorial section that the driverid schema only has two relations:
 * driverid (a String), and totmiles (a Double).
 */
val drivermileageSchema = new StructType().add("driverid",StringType,true).add("totmiles",DoubleType,true)
```

Now we can populate `drivermileageSchema` with our CSV files residing in HDFS:

```spark
%spark2
val drivermileageDataFrame = spark.read.format("csv").option("header", "true").schema(drivermileageSchema)load("hdfs:///tmp/data/drivermileage.csv")
```

Finally, let’s create a temporary view:

```spark
%spark2
drivermileageDataFrame.createOrReplaceTempView("drivermileage")
```

We can use SparkSession and SQL to query `drivermileage`:

```spark
%spark2
hiveContext.sql("SELECT * FROM drivermileage LIMIT 15").show()
```

![select-csv-from-driverm-800x348](https://user-images.githubusercontent.com/558905/54868611-9564f280-4d64-11e9-896c-660261794983.jpg)

### Query Tables To Build Spark RDD

We will do a simple select query to fetch data from geolocation and drivermileage tables to a spark variable. Getting data into Spark this way also allows to copy table schema to RDD.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Do this</h4>

```spark
%spark2
val geolocation_temp0 = hiveContext.sql("SELECT * FROM geolocation")
val drivermileage_temp0 = hiveContext.sql("SELECT * FROM drivermileage")
```

Now let’s register temporary global tables from our dataFrames and use SQL syntax to query against that table.

```spark
%spark2
geolocation_temp0.createOrReplaceTempView("geolocation_temp0")
drivermileage_temp0.createOrReplaceTempView("drivermileage_temp0")
hiveContext.sql("SHOW TABLES").show()
```

![another-way-to-make-rdd-800x426](https://user-images.githubusercontent.com/558905/54868595-9433c580-4d64-11e9-90d2-5eb00f83f79c.jpg)

### Querying Against Registered Temporary Tables

Next, we will perform an iteration and a filter operation. 

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. First, we need to filter drivers that have non-normal events associated with them and then count the number for non-normal events for each driver.</h4>

```spark
%spark2
val geolocation_temp1 = hiveContext.sql("SELECT driverid, COUNT(driverid) occurance from geolocation_temp0 WHERE event!='normal' GROUP BY driverid")
/**
 * Show RDD
 */
geolocation_temp1.show(10)
```

![filter-abnormal-events-800x78](https://user-images.githubusercontent.com/558905/54868603-94cc5c00-4d64-11e9-8486-0f74707a431f.jpg)

As stated earlier about RDD transformations, select operation is a RDD transformation and therefore does not return anything.

The resulting table will have a count of total non-normal events associated with each driver. Register this filtered table as a temporary table so that subsequent SQL queries can be applied to it:


```spark
%spark2
geolocation_temp1.createOrReplaceTempView("geolocation_temp1")
hiveContext.sql("SHOW TABLES").show()
```

![filtered-table-800x187](https://user-images.githubusercontent.com/558905/54868604-94cc5c00-4d64-11e9-914c-fa12ed923bf1.jpg)


You can view the result by executing an action operation on the temporary view.

```spark
%spark2
hiveContext.sql("SELECT * FROM geolocation_temp1 LIMIT 15").show()
```

![view-operation-800x456](https://user-images.githubusercontent.com/558905/54868615-9564f280-4d64-11e9-9ced-12aca00b099f.jpg)

### Perform join Operation

In this section we will perform a join operation geolocation_temp1 table has details of drivers and count of their respective non-normal events. The `drivermileage_temp0` table has details of total miles travelled by each driver.

We will join two tables on common column, which in our case is `driverid`:

```spark
%spark2
val joined = hiveContext.sql("select a.driverid,a.occurance,b.totmiles from geolocation_temp1 a,drivermileage_temp0 b where a.driverid=b.driverid")
```

![join_op_column_hello_hdp_lab4-800x59](https://user-images.githubusercontent.com/558905/54868607-94cc5c00-4d64-11e9-9dea-ad3f2f24d829.png)

The resulting data set will give us total miles and total non-normal events for a particular driver. Register this filtered table as a temporary table so that subsequent SQL queries can be applied to it:

```spark
%spark2
joined.createOrReplaceTempView("joined")
hiveContext.sql("SHOW TABLES").show()
```

![created-joined-800x456](https://user-images.githubusercontent.com/558905/54868598-9433c580-4d64-11e9-8f2e-c4e48b6b60e0.jpg)

You can view the result by executing action operation on our temporary view:

```spark
%spark2
/**
 * We can view the result from our query with a select statement
 */
hiveContext.sql("SELECT * FROM joined LIMIT 10").show()
```

![show-results-joined-800x456](https://user-images.githubusercontent.com/558905/54868614-9564f280-4d64-11e9-98d0-54df0e3d2b0d.jpg)

### Compute Driver Risk Factor

In this section we will associate a driver risk factor with every driver. The risk factor for each driver is the number of abnormal occurrences over the total number of miles driver. Simply put, a high number of abnormal occurrences over a short amount of miles driven is an indicator of high risk. 

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Let’s translate this intuition into an SQL query:</h4>

```spark
%spark2
val risk_factor_spark = hiveContext.sql("SELECT driverid, occurance, totmiles, totmiles/occurance riskfactor FROM joined")
```

![calculate_riskfactor_hello_hdp_lab4-800x59](https://user-images.githubusercontent.com/558905/54868596-9433c580-4d64-11e9-8e11-f48105861f91.png)


The resulting data set will give us total miles and total non-normal events and what is a risk for a particular driver. Register this filtered table as a temporary table so that subsequent SQL queries can be applied to it:

```spark
%spark2
risk_factor_spark.createOrReplaceTempView("risk_factor_spark")
hiveContext.sql("SHOW TABLES").show()
```

View the results:

```spark
%spark2
risk_factor_spark.show(10)
```

![results-from-risk-800x403](https://user-images.githubusercontent.com/558905/54868610-9564f280-4d64-11e9-8a17-053df158d7e6.jpg)

### Save Table as CSV

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. After finding the risk factor for each driver we might want to store our results as a CSV on HDFS:</h4>

```spark
%spark2
risk_factor_spark.coalesce(1).write.csv("hdfs:///tmp/data/riskfactor")
```

There will be a directory structure with our data under user/maria_dev/data/ named riskfactor there we can find our csv file with a auto generated name given to it by Spark.


### Full Spark Code Review

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Instantiate SparkSession</h4>

```spark
%spark2
val hiveContext = new org.apache.spark.sql.SparkSession.Builder().getOrCreate()
```

Shows tables in the default Hive database:

```spark
hiveContext.sql("SHOW TABLES").show()
```

Select all rows and columns from tables, stores Hive script into variable and registers variables as RDD:

```spark
val geolocation_temp0 = hiveContext.sql("SELECT * FROM geolocation")

val drivermileage_temp0 = hiveContext.sql("SELECT * FROM drivermileage")

geolocation_temp0.createOrReplaceTempView("geolocation_temp0")
drivermileage_temp0.createOrReplaceTempView("drivermileage_temp0")

val geolocation_temp1 = hiveContext.sql("SELECT driverid, count(driverid) occurance FROM geolocation_temp0 WHERE event!='normal' GROUP BY driverid")

geolocation_temp1.createOrReplaceTempView("geolocation_temp1")
```

Load first 15 rows from geolocation_temp2, which is the data from drivermileage table:

```spark
hiveContext.sql("SELECT * FROM geolocation_temp1 LIMIT 15").show()
```

Create joined to join 2 tables by the same driverid and register joined as a RDD:

```spark
val joined = hiveContext.sql("SELECT a.driverid,a.occurance,b.totmiles FROM geolocation_temp1 a,drivermileage_temp0 b WHERE a.driverid=b.driverid")
joined.createOrReplaceTempView("joined")
```

Load first 10 rows and columns in joined:

```spark
hiveContext.sql("SELECT * FROM joined LIMIT 10").show()
```

Initialize risk_factor_spark and register as an RDD:

```spark
val risk_factor_spark = hiveContext.sql("SELECT driverid, occurance, totmiles, totmiles/occurance riskfactor from joined")

risk_factor_spark.createOrReplaceTempView("risk_factor_spark")
```

Print the first 15 lines from the risk_factor_spark table:

```spark
hiveContext.sql("SELECT * FROM risk_factor_spark LIMIT 15").show()
```

### Results

Let’s summarize the Spark coding skills and knowledge we acquired to compute the risk factor associated with every driver. Spark is efficient for computation because of its in-memory data processing engine. We learned how to integrate Hive with Spark by creating a Hive Context. We used our existing data from Hive to create an RDD. We learned to perform RDD transformations and actions to create new datasets from existing RDDs. These new datasets include filtered, manipulated and processed data. After we computed risk factor, we learned to load and save data into Hive as ORC.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>