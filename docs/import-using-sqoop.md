## Lab: Sqoop Import Data From RDBMS to HDFS

A tool, which we use for importing tables from RDBMS to HDFS is the Sqoop Import tool. Here each row in a table is considered as a record in HDFS. 
Moreover, when we talk about text files all records are stored as text data. Whereas when we talk about Avro and sequence files all records are stored as binary data here. 
We can say the Sqoop Import all tables as individual tables from RDBMS to HDFS.

To import data into HDFS we use the following syntax for importing in Sqoop. Such as:

	$ sqoop import (generic-args) (import-args)

Uploading our data into files in HDFS is Sqoop’s import tool’s main function. However, if we have a Hive metastore associated with our HDFS cluster, Sqoop can also import the data into Hive. It is possible by generating and executing a CREATE TABLE statement to define the data’s layout in Hive. Also, it is the very simple method to import data into Hive, like adding the –hive-import option to your Sqoop command line.
There are two file formats in which we can import data. One is delimited text or other is SequenceFiles.

The very advantage is we can type the sqoop import arguments in any order with respect to one another. However, when it comes to the Hadoop generic arguments, those must precede any import arguments only.

First, here all the arguments are grouped into collections which are organized by function. However, some collections are present in several tools here. For example, the “common” arguments.

| Argument	| Description |
| ---- | ----|
|–connect <jdbc-uri> | Specify JDBC connect string |
|–connection-manager <class-name> | Specify connection manager class to use |
|–driver <class-name> | Manually specify JDBC driver class to use |
|–hadoop-mapped-home <dir> | Override $HADOOP_MAPRED_HOME |
|–help	| Print usage instructions |
|–password-file	| Set path for a file containing the authentication password |
|-P | Read password from console |
|–password <password> | Set authentication password |
|–username <username> | Set authentication username |
|–verbose | Print more information while working |
|–connection-param-file <filename> | Optional properties file that provides connection parameters |
|–relaxed-isolation | Set connection transaction isolation to read uncommitted for the mappers |

### Connecting to a Database Server

Sqoop is designed to import tables from a database into HDFS. To do so, you must specify a connect string that describes how to connect to the database. The connect string is similar to a URL, and is communicated to Sqoop with the –connect argument. That defines the server and database to connect to; also specify the port.

For example:

	$ sqoop import –connect jdbc:mysql://[YOUR SERVER NAME]/employees

### Sqoop Import – Validation arguments More Details

| Argument	| Description |
| ---- | ----|
| –validate | Enable validation of data copied, supports single table copy only |
| –validator <class-name> | Specify validator class to use |
| –validation-threshold <class-name> | Specify validation threshold class to use |
| –validation-failurehandler <class-name> | Specify validation failure handler class to use |

### Sqoop Import – Import control arguments

| Argument	| Description |
| ---- | ----|
| –append | Append data to an existing dataset in HDFS |
| –boundary-query <statement>	| Boundary query to use for creating splits |
| –columns <col,col,col…>	| Columns to import from table |
| –fetch-size <n>	| Number of entries to read from database at once |
| -m,–num-mappers <n>	| Use n map tasks to import in parallel |
| -e,–query <statement>	| Import the results of statement |
| –split-by <column-name>	| Column of the table used to split work units. Cannot be used with –autoreset-to-one-mapperoption |
| –table <table-name>	| Table to read |
| –target-dir <dir>	| HDFS destination dir |
| –warehouse-dir <dir>	| HDFS parent for table destination |
| –where <where clause>	| WHERE clause to use during import |
| -z,–compress	| Enable compression |
| –compression-codec <c>	| Use Hadoop codec (default gzip) |


### Selecting the Data to Import

Sqoop imports data in a table-centric fashion. we generally use the –table argument while selecting the table to import. like, –table employees. However, this argument in a database can also identify a VIEW or other table-like entity.

However, all the data is written to HDFS in its “natural order”. That is a table containing columns A, B, and C results in an import of data in Sqoop. Such as:

	A1,B1,C1
	A2,B2,C2
	…

By selecting a subset of columns, with –columns argument we can control their ordering. The only condition is that it should include a comma-delimited list of columns to import. Like: –columns “name,employee_id,jobtitle”.

### Free-form Query Imports

We can also import the result set of an arbitrary SQL query in Sqoop. Also, we can specify a SQL statement with the –query argument. Despite using the –table, –columns and –where arguments.

While we import a free-form query, we need to specify a destination directory with –target-dir.

In addition, we can import the results of a query in parallel. Afterwards,  each map task will need to execute a copy of the query, with results partitioned by bounding conditions inferred by Sqoop. However, our query must include the token $CONDITIONS. That each Sqoop process will replace with a unique condition expression. Also important to select a splitting column with –split-by.

For example:

	$ sqoop import \
		–query ‘SELECT a.*, b.* FROM a JOIN b on (a.id == b.id) WHERE $CONDITIONS’ \
		–split-by a.id –target-dir /user/foo/joinresults

By specifying a single map task with -m 1, the query can be executed once and imported serially.

	$ sqoop import \
		–query ‘SELECT a.*, b.* FROM a JOIN b on (a.id == b.id) WHERE $CONDITIONS’ \
		-m 1 –target-dir /user/foo/joinresults

### Controlling Parallelism

From most database sources, Sqoop imports data in parallel. Also, to perform the import in sqoop by using the -m or –num-mappers argument. Moreover, we can specify the number of map tasks (parallel processes) to use each of these arguments takes an integer value which corresponds to the degree of parallelism to employ. 

### Controlling type mapping

To map most SQL types to appropriate Java or Hive representatives, Sqoop is preconfigured. Although, here also the default mapping might not be suitable for everyone. Also, might be overridden. Either by –map-column-java (for changing the mapping to Java) or –map-column-hive (for changing Hive mapping).
Sqoop is expecting the comma-separated list of mapping in the form <name of column>=<new type>. For example:

	$ sqoop import … –map-column-java id=String,value=Integer

Also, Sqoop will raise the exception in case that some configured mapping will not be used.

### Additional Import Configuration Properties

Some additional properties which can be configured by modifying conf/sqoop-site.xml. However, Properties can be specified the same as in Hadoop configuration files.

For example:

```xml
<property>
	<name>property.name</name>
	<value>property.value</value>
</property>
```
	
On the command line in the generic arguments, they can also be specified.  For example:

	$ sqoop import -D property.name=property.value …

	
### Sqoop Import – Hive arguments

| Argument	| Description |
| ---- | ----|
| –hive-import	| Import tables into Hive (Uses Hive’s default delimiters if none are set.) |
| –hive-overwrite	| Overwrite existing data in the Hive table |
| –create-hive-table	| If set, then the job will fail if the target hive table exits. By default this property is false |
| –hive-table <table-name>	| Sets the table name to use when importing to Hive |
| –hive-drop-import-delims	| Drops \n, \r, and \01 from string fields when importing to Hive |
| –hive-delims-replacement	| Replace \n, \r, and \01 from string fields with user defined string when importing to Hive |
| –hive-partition-key	| Name of a hive field to partition are sharded on |
| –hive-partition-value <v>	| String-value that serves as partition key for this imported into hive in this job |
| –map-column-hive <map>	| Override default mapping from SQL type to Hive type for configured columns |


We will understand how to use the import tool in a variety of situations by the following examples.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>1. Put the Data in HDFS</h4>

In addition, a basic import of a table named EMPLOYEES in the corp database:

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES

Also, a basic import requiring a login:

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES \
	–username SomeUser -P
	Enter password: (hidden)

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>2. Select the Data to import into HDFS</h4>

So selecting specific columns from the EMPLOYEES table:

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES \
		–columns “employee_id,first_name,last_name,job_title”

Controlling the import parallelism (using 4 parallel tasks):

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES -m 4

Also, specifying the delimiters to use in a text-mode import:

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES \
		–fields-terminated-by ‘\t’ –lines-terminated-by ‘\n’ \
		–optionally-enclosed-by ‘\”‘

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>3. Import the Data into Hive</h4>

Here, importing the data to Hive:

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES –hive-import

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>4. Import partial Data into Hive</h4>

Now, here, only import new employees:

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES –hive-import \
		–where “start_date > ‘2010-01-01′”

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>5. Split the Data into Hive</h4>

And, changing the splitting column from the default:

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES –hive-import \
		–split-by dept_id

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>6. Verify the Data</h4>

Now, let's verify that an import was successful:

	$ hdfs dfs -ls EMPLOYEES
	Found 5 items
	drwxr-xr-x   – someuser somegrp          0 2010-04-27 16:40 /user/someuser/EMPLOYEES/_logs
	-rw-r–r–   1 someuser somegrp    2913511 2010-04-27 16:40 /user/someuser/EMPLOYEES/part-m-00000
	-rw-r–r–   1 someuser somegrp    1683938 2010-04-27 16:40 /user/someuser/EMPLOYEES/part-m-00001
	-rw-r–r–   1 someuser somegrp    7245839 2010-04-27 16:40 /user/someuser/EMPLOYEES/part-m-00002
	-rw-r–r–   1 someuser somegrp    7842523 2010-04-27 16:40 /user/someuser/EMPLOYEES/part-m-00003

	$ hadoop fs -cat EMPLOYEES/part-m-00000 | head -n 10
	0,joe,smith,engineering
	1,jane,doe,marketing
	…

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>7. Incrementally Import new Data</h4>

After having already imported the first 100,000 rows of a table, Here performing an incremental import of new data:

	$ sqoop import –connect jdbc:mysql://[your server name]/somedb –table sometable –hive-import \
		–where “id > 100000” –target-dir /incremental_dataset –append

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h4>8. Validate the new Data</h4>

In the corp database, there is an import of a table named EMPLOYEES. That uses validation to validate the import. By using the table row count and the number of rows copied into HDFS. 

	$ sqoop import –connect jdbc:mysql://[your server name]/corp –table EMPLOYEES –validate

### Results

In this article, we have learned the whole concept of Sqoop Import. Also, we have seen various Sqoop Import examples and Sqoop import syntax. However, if you want to ask any query regarding, please ask in the comment section below. We will definitely get back to you.

<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>