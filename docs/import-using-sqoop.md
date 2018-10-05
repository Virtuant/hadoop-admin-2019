Sqoop Import- Importing Data From RDBMS to HDFS
1 Feb, 2018  in Sqoop Tutorials by Data Flair

https://data-flair.training/blogs/sqoop-import/

Contents

    1. Sqoop Import – Objective
    2. Sqoop Import and its Purpose
    3. Sqoop Import Syntax
    4. Sqoop Import Example Invocations
    5. Conclusion

1. Sqoop Import – Objective

In the last article, we discussed Sqoop Export. In this article we will discuss Sqoop import, a tool which we use for importing tables from RDBMS to HDFS is the Sqoop Import tool. Here, we will learn how to Sqoop import multiple tables from RDBMS database to Hadoop HDFS. Moreover, we will learn the purpose of importing in Sqoop, Sqoop import syntax as well as Sqoop import query examples to understand it well.
Introduction to Sqoop Import

Sqoop Import – Introduction
2. Sqoop Import and its Purpose

A tool, which we use for importing tables from RDBMS to HDFS is the Sqoop Import tool. Basically, here each row in a table is considered as a record in HDFS. Moreover, when we talk about text files all records are stored as text data. Whereas when we talk about Avro and sequence files all records are stored as binary data here. Basically, we can say the Sqoop Import all tables as individual tables from RDBMS to HDFS.

Let’s discuss HDFS Features
3. Sqoop Import Syntax

To import data into HDFS we use the following syntax for importing in Sqoop. Such as:

$ sqoop import (generic-args) (import-args)

$ sqoop-import (generic-args) (import-args)

The very advantage is we can type the sqoop import arguments in any order with respect to one another. However, when it comes to the Hadoop generic arguments, those must precede any import arguments only.

Basically, here all the arguments are grouped into collections which are organized by function. However, some collections are present in several tools here. For example, the “common” arguments.

Table 1. Sqoop Import – Common arguments
Argument	Description
–connect <jdbc-uri>	Specify JDBC connect string
–connection-manager <class-name>	Specify connection manager class to use
–driver <class-name>	Manually specify JDBC driver class to use
–hadoop-mapped-home <dir>	Override $HADOOP_MAPRED_HOME
–help	Print usage instructions
–password-file	Set path for a file containing the authentication password
-P	Read password from console
–password <password>	Set authentication password
–username <username>	Set authentication username
–verbose	Print more information while working
–connection-param-file <filename>	Optional properties file that provides connection parameters
–relaxed-isolation	Set connection transaction isolation to read uncommitted for the mappers.

a. Connecting to a Database Server

Sqoop is designed to import tables from a database into HDFS. To do so, you must specify a connect string that describes how to connect to the database. The connect string is similar to a URL, and is communicated to Sqoop with the –connect argument. That defines the server and database to connect to; also specify the port.

For example:

$ sqoop import –connect jdbc:mysql://database.example.com/employees

Table 2. Sqoop Import – Validation arguments More Details
Argument	Description
–validate	Enable validation of data copied, supports single table copy only.
–validator <class-name>	Specify validator class to use.
–validation-threshold <class-name>	Specify validation threshold class to use.
–validation-failurehandler <class-name>	Specify validation failure handler class to use.

Table 3. Sqoop Import – Import control arguments
Argument	Description
–append	Append data to an existing dataset in HDFS
–as-avrodatafile	Imports data to Avro Data Files
–as-sequencefile	Imports data to SequenceFiles
–as-textfile	Imports data as plain text (default)
–as-parquetfile	Imports data to Parquet Files
–boundary-query <statement>	Boundary query to use for creating splits
–columns <col,col,col…>	Columns to import from table
–delete-target-dir	Delete the import target directory if it exists
–direct	Use direct connector if exists for the database
–fetch-size <n>	Number of entries to read from database at once.
–inline-lob-limit <n>	Set the maximum size for an inline LOB
-m,–num-mappers <n>	Use n map tasks to import in parallel
-e,–query <statement>	Import the results of statement.
–split-by <column-name>	Column of the table used to split work units. Cannot be used with –autoreset-to-one-mapperoption.
–autoreset-to-one-mapper	Import should use one mapper if a table has no primary key and no split-by column is provided. Cannot be used with –split-by <col> option.
–table <table-name>	Table to read
–target-dir <dir>	HDFS destination dir
–warehouse-dir <dir>	HDFS parent for table destination
–where <where clause>	WHERE clause to use during import
-z,–compress	Enable compression
–compression-codec <c>	Use Hadoop codec (default gzip)
–null-string <null-string>	The string to be written for a null value for string columns
–null-non-string <null-string>	The string to be written for a null value for non-string columns

Although, both –null-string and –null-non-string arguments are optional.However, we use the string “null” if not specified.

b. Selecting the Data to Import

Basically, Sqoop imports data in a table-centric fashion. we generally use the –table argument while selecting the table to import. like, –table employees. However, this argument in a database can also identify a VIEW or other table-like entity.

However, all the data is written to HDFS in its “natural order”. That is a table containing columns A, B, and C results in an import of data in Sqoop. Such as:

A1,B1,C1

A2,B2,C2

…

By selecting a subset of columns, with –columns argument we can control their ordering. The only condition is that it should include a comma-delimited list of columns to import. Like: –columns “name,employee_id,jobtitle”.

c. Free-form Query Imports

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

d. Controlling Parallelism

From most database sources, Sqoop imports data in parallel. Also, to perform the import in sqoop by using the -m or –num-mappers argument. Moreover, we can specify the number of map tasks (parallel processes) to use each of these arguments takes an integer value which corresponds to the degree of parallelism to employ. 

e. Controlling Distributed Cache

Basically, in $SQOOP_HOME/lib folder, Sqoop will copy the jars to job cache every time when starting a Sqoop job. However,  when  Oozie launched it, this is unnecessary since Oozie uses its own Sqoop share lib which keeps Sqoop dependencies in the distributed cache. Although, for the Sqoop dependencies Oozie will do the localization on each worker node only once during the first Sqoop job. Also, reuse the jars on worker node for subsequential jobs. 

f. Controlling the Sqoop Import Process

The import process in sqoop will use JDBC, by default. That provides a reasonable cross-vendor import channel. However, by using database-specific data movement tools, some databases can perform imports in a more high-performance fashion.

In addition, inside your home directory in HDFS, Sqoop will import a table named foo to a directory named foo. For example, the Sqoop import tool will write to /user/someuser/foo/(files) if your username is some user. However, we can adjust the parent directory of the import with the –warehouse-dir argument. For example:

$ sqoop import –connnect <connect-str> –table foo –warehouse-dir /shared \

   …

g. Controlling transaction isolation

Basically, to import data the read committed transaction isolation in the mappers are used in Sqoop. In all ETL workflows, this may not be the ideal. Yet it may desire to reduce the isolation guarantees. However, to instruct Sqoop to use read uncommitted isolation level we can use the –relaxed-isolation option.

Although, on all databases, the read-uncommitted isolation level is not supported. For example, Oracle. So specifying the option –relaxed-isolation may not be supported on all databases.

h. Controlling type mapping

Basically, to map most SQL types to appropriate Java or Hive representatives, Sqoop is preconfigured. Although, here also the default mapping might not be suitable for everyone. Also, might be overridden. Either by –map-column-java (for changing the mapping to Java) or –map-column-hive (for changing Hive mapping).

Table 4. Parameters for overriding mapping
Argument	Description
–map-column-java <mapping>	Override mapping from SQL to Java type for configured columns.
–map-column-hive <mapping>	Override mapping from SQL to Hive type for configured columns.

Basically, Sqoop is expecting the comma-separated list of mapping in the form <name of column>=<new type>. For example:

$ sqoop import … –map-column-java id=String,value=Integer

Also, Sqoop will raise the exception in case that some configured mapping will not be used.

i. Incremental Imports

There is an incremental import mode offered by Sqoop. That can be used to retrieve only rows newer than some previously imported set of rows.

The following arguments control incremental imports in sqoop:

Table 5. Sqoop Import – Incremental import arguments
Argument	Description
–check-column (col)	Specifies the column to be examined when determining which rows to import. (the column should not be of type CHAR/NCHAR/VARCHAR/VARNCHAR/ LONGVARCHAR/LONGNVARCHAR)
–incremental (mode)	Specifies how Sqoop determines which rows are new. Legal values for mode include append and lastmodified.
–last-value (value)	Specifies the maximum value of the check column from the previous import.

Basically, there are two types of incremental imports in Sqoop.One is appended and second is last modified. Moreover, to specify the type of incremental import to perform, we can also use the –incremental argument.

j. File Formats

Basically, there are two file formats in which we can import data. One is delimited text or other is SequenceFiles.

k. Large Objects

In particular ways, Sqoop handles large objects (BLOB and CLOB columns). However, if this data is truly large, then these columns should not be fully materialized in memory for manipulation, as most columns are. Despite, their data is handled in a streaming fashion. 

Table 6. Sqoop Import – Output line formatting arguments
Argument	Description
–enclosed-by <char>	Sets a required field enclosing character
–escaped-by <char>	Sets the escape character
–fields-terminated-by <char>	Sets the field separator character
–lines-terminated-by <char>	Sets the end-of-line character
–mysql-delimiters	Uses MySQL’s default delimiter set: fields: , lines: \n escaped-by: \ optionally-enclosed-by: ‘
–optionally-enclosed-by <char>	Sets a field enclosing character

Table 7. Sqoop Import – Input parsing arguments
Argument	Description
–input-enclosed-by <char>	Sets a required field encloser
–input-escaped-by <char>	Sets the input escape character
–input-fields-terminated-by <char>	Sets the input field separator
–input-lines-terminated-by <char>	Sets the input end-of-line character
–input-optionally-enclosed-by <char>	Sets a field enclosing character

Table 8. Sqoop Import – Hive arguments
Argument	Description
–hive-home <dir>	Override $HIVE_HOME
–hive-import	Import tables into Hive (Uses Hive’s default delimiters if none are set.)
–hive-overwrite	Overwrite existing data in the Hive table.
–create-hive-table	If set, then the job will fail if the target hive table exits. By default this property is false.
	
–hive-table <table-name>	Sets the table name to use when importing to Hive.
–hive-drop-import-delims	Drops \n, \r, and \01 from string fields when importing to Hive.
–hive-delims-replacement	Replace \n, \r, and \01 from string fields with user defined string when importing to Hive.
–hive-partition-key	Name of a hive field to partition are sharded on
–hive-partition-value <v>	String-value that serves as partition key for this imported into hive in this job.
–map-column-hive <map>	Override default mapping from SQL type to Hive type for configured columns.

l. Importing Data Into Hive

Uploading our data into files in HDFS is Sqoop’s import tool’s main function. However, if we have a Hive metastore associated with our HDFS cluster, Sqoop can also import the data into Hive. It is possible by generating and executing a CREATE TABLE statement to define the data’s layout in Hive. Also, it is the very simple method to import data into Hive, like adding the –hive-import option to your Sqoop command line.

Table 9. Sqoop Import – HBase arguments
Argument	Description
–column-family <family>	Sets the target column family for the import
–hbase-create-table	If specified, create missing HBase tables
–hbase-row-key <col>	Specifies which input column to use as the row key. In case, if input table contains composite key, then <col> must be in the form of a comma-separated list of composite key attributes.
–hbase-table <table-name>	Specifies an HBase table to use as the target instead of HDFS
–hbase-bulkload	Enables bulk loading

m. Importing Data Into HBase

Beyond HDFS and Hive, Sqoop supports additional import targets. Like Sqoop can also import records into a table in HBase.

Table 10. Sqoop Import – Accumulo arguments
Argument	Description
–accumulo-table <table-nam>	Specifies an Accumulo table to use as the target instead of HDFS
–accumulo-column-family <family>	Sets the target column family for the import
–accumulo-create-table	If specified, create missing Accumulo tables
–accumulo-row-key <col>	Specifies which input column to use as the row key
–accumulo-visibility <vis>	(Optional) Specifies a visibility token to apply to all rows inserted into Accumulo. Default is the empty string.
–accumulo-batch-size <size>	(Optional) Sets the size in bytes of Accumulo’s write buffer. Default is 4MB.
–accumulo-max-latency <ms>	(Optional) Sets the max latency in milliseconds for the Accumulo batch writer. Default is 0.
–accumulo-zookeepers <host:port>	Comma-separated list of Zookeeper servers used by the Accumulo instance
–accumulo-instance <table-name>	Name of the target Accumulo instance
–accumulo-user <username>	Name of the Accumulo user to import as
–accumulo-password <password>	Password for the Accumulo user

n. Importing Data Into Accumulo

Also, in Accumulo, Sqoop supports importing records into a table.

Table 11. Sqoop Import – Code generation arguments
Argument	Description
–bindir <dir>	Output directory for compiled objects
–class-name <name>	Sets the generated class name. This overrides –package-name. When combined with –jar-file, sets the input class.
–jar-file <file>	Disable code generation; use specified jar
–outdir <dir>	Output directory for generated code
–package-name <name>	Put auto-generated classes in this package
–map-column-java <m>	Override default mapping from SQL type to Java type for configured columns.

o. Additional Import Configuration Properties

Some additional properties which can be configured by modifying conf/sqoop-site.xml. However, Properties can be specified the same as in Hadoop configuration files.

For example:

<property>

<name>property.name</name>

<value>property.value</value>

</property>

On the command line in the generic arguments, they can also be specified.  For example:

sqoop import -D property.name=property.value …

Table 12. Sqoop Import – Additional import configuration properties
Argument	Description
sqoop.bigdecimal.format.string	Controls how BigDecimal columns will formatted when stored as a String. A value of true (default) will use toPlainString to store them without an exponent component (0.0000001); while a value of false will use toString which may include an exponent (1E-7)
sqoop.hbase.add.row.key	When set to false (default), Sqoop will not add the column used as a row key into the row data in HBase. When set to true, the column used as a row key will be added to the row data in HBase.
4. Sqoop Import Example Invocations

Basically, we will understand how to use the import tool in a variety of situations by the following examples.

In addition, a basic import of a table named EMPLOYEES in the corp database:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES

Also, a basic import requiring a login:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

–username SomeUser -P

Enter password: (hidden)

So selecting specific columns from the EMPLOYEES table:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

–columns “employee_id,first_name,last_name,job_title”

Controlling the import parallelism (using 8 parallel tasks):

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

   -m 8

Storing data in SequenceFiles, and setting the generated class name to com.foocorp.Employee:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

–class-name com.foocorp.Employee –as-sequencefile

Also, specifying the delimiters to use in a text-mode import:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

–fields-terminated-by ‘\t’ –lines-terminated-by ‘\n’ \

–optionally-enclosed-by ‘\”‘

Basically here, importing the data to Hive:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

–hive-import

Also, here, only importing new employees:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

–where “start_date > ‘2010-01-01′”

Afterwards, changing the splitting column from the default:

$ sqoop import –connect jdbc:mysql://db.foo.com/corp –table EMPLOYEES \

–split-by dept_id

Then, we are verifying that an import was successful:

$ hadoop fs -ls EMPLOYEES

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

After having already imported the first 100,000 rows of a table, Here performing an incremental import of new data:

$ sqoop import –connect jdbc:mysql://db.foo.com/somedb –table sometable \

–where “id > 100000” –target-dir /incremental_dataset –append

In the corp database, there is an import of a table named EMPLOYEES. That uses validation to validate the import. By using the table row count and the number of rows copied into HDFS. 

$ sqoop import –connect jdbc:mysql://db.foo.com/corp \

–table EMPLOYEES –validate
5. Conclusion

Hence, in this article, we have learned the whole concept of Sqoop Import. Also, we have seen various Sqoop Import examples and Sqoop import syntax. However, if you want to ask any query regarding, please ask in the comment section below. We will definitely get back to you.
