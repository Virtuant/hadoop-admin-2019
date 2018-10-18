## DataFrame and Datasets in Spark REPL

Spark SQl is a Spark module for structured data processing. It has interfaces that provide Spark with additional information about the structure of both the data and the computation being performed. The additional information is used for optimization.

* A Dataset is a type of interface that provides the benefits of RDD (strongly typed) and Spark SQL's optimization. It is important to note that a Dataset can be constructed from JVM objects and then manipulated using complex functional transformations, however, they are beyond this quick guide.
* A DataFrame is a Dataset organized into named columns. Conceptually, they are equivalent to a table in a relational database or a DataFrame in R or Python. DataFrames can be created from various sources such as:
Key difference between the Dataset and the DataFrame is that Datasets are strongly typed.

----

### Download the Dataset

In preparation for this tutorial you need to download two files, people.txt and people.json into your  Sandbox's **tmp** folder. The commands below should be typed into [Shell-in-a-Box](sandbox-hdp.hortonworks.com:4200)

1\. Go to the `/tmp` directory:

~~~ bash
cd /tmp
~~~

2\. Copy and paste the command to download the `people.txt`:

~~~bash
wget https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/dataFrame-and-dataset-examples-in-spark-repl/assets/people.txt
~~~

3\. Copy and paste the command to download the `people.json`:

~~~bash
wget https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/dataFrame-and-dataset-examples-in-spark-repl/assets/people.json
~~~

### Upload the dataset to HDFS

1\. Before moving the files into HDFS you need to login under **hdfs** user in order to give permission to perform file operations:

~~~bash
su hdfs
cd
~~~

2\. Next, upload people.txt and people.json files to HDFS:

~~~bash
#Copy files from local system to HDF
hdfs dfs -put /tmp/people.txt /tmp/people.txt
hdfs dfs -put /tmp/people.json /tmp/people.json
~~~

3\.Verify that both files were copied into HDFS **/tmp** folder by copying the following commands:

~~~bash
#Verify that files were move to HDF
hdfs dfs -ls /tmp
~~~

4\.Launch the Spark Shell:

~~~ bash
spark-shell
~~~

## DataFrame API

DataFrame API provides easier access to data since it looks conceptually like a Table and a lot of developers from Python/R/Pandas are familiar with it.

At a `scala>` REPL prompt, type the following:

~~~scala
val df = spark.read.json("/tmp/people.json")
~~~

Using `df.show`, display the contents of the DataFrame:

~~~scala
df.show
~~~

You should see an output similar to:

~~~scala
...
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+

scala>
~~~

### Additional DataFrame API

Now, lets select "name" and "age" columns and increment the "age" column by 1:

~~~scala
df.select(df("name"), df("age") + 1).show()
~~~

This will produce an output similar to the following:

~~~ bash
...
+-------+---------+
|   name|(age + 1)|
+-------+---------+
|Michael|     null|
|   Andy|       31|
| Justin|       20|
+-------+---------+

scala>
~~~

To return people older than 21, use the filter() function:

~~~scala
df.filter(df("age") > 21).show()
~~~

This will produce an output similar to the following:

~~~ bash
...
+---+----+
|age|name|
+---+----+
| 30|Andy|
+---+----+

scala>
~~~

Next, to count the number of people of specific age, use groupBy() & count() functions:

~~~scala
df.groupBy("age").count().show()
~~~

This will produce an output similar to the following:

~~~ bash
...
+----+-----+
| age|count|
+----+-----+
|  19|    1|
|null|    1|
|  30|    1|
+----+-----+

scala>
~~~

### Programmatically Specifying Schema

Type the following commands(one line a time) into your Spark-shell:

1\. Import the necessary libraries

~~~scala
import org.apache.spark.sql._
import org.apache.spark.sql.Row
import org.apache.spark.sql.types._
import spark.implicits._
~~~

2\. Create and RDD

~~~scala
val peopleRDD = spark.sparkContext.textFile("/tmp/people.txt")
~~~

3\. Encode the Schema in a string

~~~scala
val schemaString = "name age"
~~~

4\. Generate the schema based on the string of schema

~~~scala
val fields = schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, nullable = true))

val schema = StructType(fields)
~~~

5\. Convert records of the RDD (people) to Rows

~~~scala
val rowRDD = peopleRDD.map(_.split(",")).map(attributes => Row(attributes(0), attributes(1).trim))
~~~

6\.  Apply the schema to the RDD

~~~scala
val peopleDF = spark.createDataFrame(rowRDD, schema)
~~~

6\. Creates a temporary view using the DataFrame

~~~scala
peopleDF.createOrReplaceTempView("people")
~~~

7\. SQL can be run over a temporary view created using DataFrames

~~~scala
val results = spark.sql("SELECT name FROM people")
~~~

8\.The results of SQL queries are DataFrames and support all the normal RDD operations. The columns of a row in the result can be accessed by field index or by field name

~~~scala
results.map(attributes => "Name: " + attributes(0)).show()
~~~

This will produce an output similar to the following:

~~~ bash
...
+-------------+
|        value|
+-------------+
|Name: Michael|
|   Name: Andy|
| Name: Justin|
+-------------+

scala>
~~~

## DataSet API

If you haven't done so already in previous sections, make sure to upload people data sets (people.txt and people.json) to HDFS: [Environment Setup](https://hortonworks.com/tutorial/dataframe-and-dataset-examples-in-spark-repl/#environment-setup) and imported the libraries in step 1 of **Programmatically Specifying Schema** above:

Finally, if you haven't already:

~~~ bash
spark-shell
~~~

The Spark Dataset API brings the best of RDD and Data Frames together, for type safety and user functions that run directly on existing JVM types.

Let's try the simplest example of creating a dataset by applying a *toDS()* function to a sequence of numbers.

At the `scala>` prompt, copy & paste the following:

~~~scala
val ds = Seq(1, 2, 3).toDS()

ds.show
~~~

You should see the following output:

~~~ bash
+-----+
|value|
+-----+
|    1|
|    2|
|    3|
+-----+
~~~

Moving on to something slightly more interesting, let's prepare a *Person* class to hold our person data. We will use it in two ways by applying it directly on a hardcoded data and then on a data read from a json file.

To apply *Person* class to hardcoded data type:

~~~ bash
case class Person(name: String, age: Long)

val ds = Seq(Person("Andy", 32)).toDS()
~~~

When you type

~~~ bash
ds.show
~~~

you should see the following output of the *ds* Dataset

~~~ bash
+----+---+
|name|age|
+----+---+
|Andy| 32|
+----+---+
~~~

Finally, let's map data read from *people.json* to a *Person* class. The mapping will be done by name.

~~~ bash
val path = "/tmp/people.json"
val people = spark.read.json(path).as[Person] // Creates a DataSet
~~~

To view contents of people DataFrame type:

~~~scala
people.show
~~~

You should see an output similar to the following:

~~~ bash
...
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~

Note that the *age* column contains a *null* value. Before we can convert our people DataFrame to a Dataset, let's filter out the *null* value first:

~~~ bash
val pplFiltered = people.filter("age is not null")
~~~

Now we can map to the *Person* class and convert our DataFrame to a Dataset.

~~~ bash
val pplDS = pplFiltered.as[Person]
~~~

View the contents of the Dataset type

~~~ bash
pplDS.show
~~~

You should see the following:

~~~ bash
+------+---+
|  name|age|
+------+---+
|  Andy| 30|
|Justin| 19|
+------+---+
~~~

To exit type:

~~~bash
:quit
~~~

### Results

Congratulations! You now know more about the role that Spark SQL, Datasets and DataFrames have in Spark. Additionally, you learned how to use the SparkSQL interface via Shell-in-a-Box and view the content of Datasets.
