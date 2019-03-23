## Phoenix on HBase

**Objective**: Begin using the Phoenix system.

**Exercise directory**: A file from Hortonworks

Phoenix takes your SQL query, compiles it into a series of HBase scans, and orchestrates the running of 
those scans to produce regular JDBC result sets. Direct use of the HBase API, along with coprocessors 
and custom filters, results in performance on the order of milliseconds for small queries, or seconds 
for tens of millions of rows.

----

## Go into HBase

Create a table and list it:

	hbase> create 'driver_dangerous_event','events'
	hbase> list

Now `quit`.

### Import the Data

Let’s import some data into the table. We’ll use a sample dataset that tracks driving record of a logistics company. Download the `data.csv` file and let’s copy the file in HDFS:

	curl -o ~/data.csv https://raw.githubusercontent.com/hortonworks/data-tutorials/d0468e45ad38b7405570e250a39cf998def5af0f/tutorials/hdp/hdp-2.5/introduction-to-apache-hbase-concepts-apache-phoenix-and-new-backup-restore-utility-in-hbase/assets/data.csv

>Note: the `~/` puts the file into your user's `/home/[user name]` directory.

Now put the file into HDFS:

	hdfs dfs -copyFromLocal ~/data.csv /tmp

Now execute the `ImportTsv` tool from hbase user statement as following:

	hbase org.apache.hadoop.hbase.mapreduce.ImportTsv 
	-Dimporttsv.separator=, 
	-Dimporttsv.columns="HBASE_ROW_KEY,events:driverId,
	events:driverName,events:eventTime,events:eventType,events:latitudeColumn,
	events:longitudeColumn,events:routeId,events:routeName,events:truckId" 
	driver_dangerous_event 
	hdfs://master1.hdp.com:/tmp/data.csv

>Note: easier copy help:

	hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=, -Dimporttsv.columns="HBASE_ROW_KEY,events:driverId,events:driverName,events:eventTime,events:eventType,events:latitudeColumn,events:longitudeColumn,events:routeId,events:routeName,events:truckId" driver_dangerous_event hdfs://master1.hdp.com:/tmp/data.csv
	
>Question: did a `mapreduce` run? Why?

Now let’s check whether the data got imported in the table `driver_dangerous_events` or not. 

Go back to the hbase shell, and do a scan:

```sql
	hbase> scan 'driver_dangerous_event'
```

You will see all the data present in the table with row keys and the different values for different columns in a column family.

### Put Data In

Using put command, you can insert rows in a HBase table like this:

	hbase> put '<table_name>','row1','<column_family:column_name>','value'

So copy following lines to put the data in the table:

```sql
	put 'driver_dangerous_event','4','events:driverId','78'
	put 'driver_dangerous_event','4','events:driverName','Carl'
	put 'driver_dangerous_event','4','events:eventTime','2016-09-23 03:25:03.567'
	put 'driver_dangerous_event','4','events:eventType','Normal'
	put 'driver_dangerous_event','4','events:latitudeColumn','37.484938'
	put 'driver_dangerous_event','4','events:longitudeColumn','-119.966284'
	put 'driver_dangerous_event','4','events:routeId','845'
	put 'driver_dangerous_event','4','events:routeName','Santa Clara to San Diego'
	put 'driver_dangerous_event','4','events:truckId','637'
```

Now let’s view a data from scan command:

	hbase> scan 'driver_dangerous_event'

You can also update an existing cell value using the put command. The syntax for replacing is same as inserting a new value.

So let’s update a route name value of row key 4, from 'Santa Clara to San Diego' to 'Santa Clara to Los Angeles'. Type the following command in HBase shell:

	hbase>put 'driver_dangerous_event','4','events:routeName','Santa Clara to Los Angeles'

Now scan the table to see the updated data:

	hbase> scan 'driver_dangerous_event'
	
you now should see something like:

```sql
hbase(main):001:0> scan 'driver_dangerous_event'
ROW    COLUMN+CELL                                                                       
 1     column=events:driverId, timestamp=1535799477495, value=123                        
 1     column=events:driverName, timestamp=1535799477495, value=James                    
 1     column=events:eventTime, timestamp=1535799477495, value=2015-08-21 12:23:45.231   
 1     column=events:eventType, timestamp=1535799477495, value=Normal                    
 1     column=events:latitudeColumn, timestamp=1535799477495, value=38.440467            
 1     column=events:longitudeColumn, timestamp=1535799477495, value=-122.714431         
 1     column=events:routeId, timestamp=1535799477495, value=345
 2     column=events:driverId, timestamp=1535799477495, value=352                        
 2     column=events:driverName, timestamp=1535799477495, value=John 
 ...
 ```

Now for a GET to retrive the details from the row 1 and the driverName of column family events:

	hbase> get 'driver_dangerous_event','1',{COLUMN => 'events:driverName'}

If you want to view the data from two columns, just add it to the {COLUMN =>…} section. Run the following command to get the details from row key 1 and the driverName and routeId of column family events:

	hbase> get 'driver_dangerous_event','1',{COLUMNS => ['events:driverName','events:routeId']}

### Phoenix Shell

To connect to Phoenix, you need to log in, it is localhost. 

To launch it, execute the following commands:

	cd /usr/hdp/current/phoenix-client/bin
	./sqlline.py localhost

You can create a Phoenix table/view on a pre-existing HBase table. There is no need to move the data to Phoenix or convert it. Apache Phoenix supports table creation and versioned incremental alterations through DDL commands. The table metadata is stored in an HBase table and versioned. 

You can either create a READ-WRITE table or a READ only view with a condition that the binary representation of the row key and key values must match that of the Phoenix data types. The only addition made to the HBase table is Phoenix coprocessors used for query processing. A table can be created with the same name.

> NOTE: The DDL used to create the table is case sensitive and if HBase table name is in lowercase, you have to put the name in between double quotes. In HBase, you don’t model the possible KeyValues or the structure of the row key. This is the information you specify in Phoenix and beyond the table and column family.

Show help screen:

	localhost> help

Create a Phoenix table from existing HBase table by writing a code like this:

	localhost> create table "driver_dangerous_event" ("row" VARCHAR primary key,"events"."driverId" VARCHAR,
	"events"."driverName" VARCHAR,"events"."eventTime" VARCHAR,"events"."eventType" VARCHAR,
	"events"."latitudeColumn" VARCHAR,"events"."longitudeColumn" VARCHAR,
	"events"."routeId" VARCHAR,"events"."routeName" VARCHAR,"events"."truckId" VARCHAR);

Describe the table:

	localhost> !describe "driver_dangerous_event"

You can view the HBase table data from this Phoenix table:

	localhost> select * from "driver_dangerous_event";

If you want to change the view from horizontal to vertical, type the following command in the shell and then try to view the data again:

	!outputformat vertical
	localhost> select * from "driver_dangerous_event";

If you do not like this view, you can change it back to horizontal view by the following:

	!outputformat table

So with all existing HBase tables, you can query them with SQL now. You can point your Business Intelligence tools and Reporting Tools and other tools which work with SQL and query HBase as if it was another SQL database with the help of Phoenix.

### Inserting Data via Phoenix

You can insert the data using UPSERT command. It inserts if not present and updates otherwise the value in the table. The list of columns is optional and if not present, the values will map to the column in the order they are declared in the schema. 

Copy the UPSERT statement given below and then view the newly added row:

```sql
	UPSERT INTO "driver_dangerous_event" values('5','23','Matt','2016-02-29 12:35:21.739','Abnormal','23.385908','-101.384927','249','San Tomas to San Mateo','814');
```

and now select the data:

```sql
	select * from "driver_dangerous_event";
```

You will see a newly added row.

Do this several times changing the data each time.

You can set several variables like this:

```sql
	!set color true
```

And again:

```sql
	select * from "driver_dangerous_event";
```

And individual commands are available:

```sql
	!tables
```

Or `rowlimit`:

```sql
0: jdbc:phoenix:localhost> !set rowlimit 1
0: jdbc:phoenix:localhost> select * from "driver_dangerous_event";
+------+-----------+-------------+--------------------------+------------+-----------------+------------------+
| row  | driverId  | driverName  |        eventTime         | eventType  | latitudeColumn  | longitudeColumn  |
+------+-----------+-------------+--------------------------+------------+-----------------+------------------+
| 1    | 123       | James       | 2015-08-21 12:23:45.231  | Normal     | 38.440467       | -122.714431      |
+------+-----------+-------------+--------------------------+------------+-----------------+------------------+
1 row selected (0.047 seconds)
```

### Create Table in Phoenix

Now let's do the same thing from the Phoenix side:

```sql
0: jdbc:phoenix:localhost> create table driver (id integer primary key, firstname varchar, lastname varchar);
No rows affected (0.758 seconds)
```

Now check it:

```sql
0: jdbc:phoenix:localhost> select * from driver;
+-----+------------+-----------+
| ID  | FIRSTNAME  | LASTNAME  |
+-----+------------+-----------+
+-----+------------+-----------+
No rows selected (0.019 seconds)
```

Of course you can also do it this way:

	!tables

In hbase the table looks like this:

```sql
hbase(main):022:0> describe 'DRIVER'
Table DRIVER is ENABLED                                                                                        
DRIVER, {TABLE_ATTRIBUTES => {coprocessor$1 => '|org.apache.phoenix.coprocessor.ScanRegionObserver|805306366|',
 coprocessor$2 => '|org.apache.phoenix.coprocessor.UngroupedAggregateRegionObserver|805306366|', coprocessor$3 
=> '|org.apache.phoenix.coprocessor.GroupedAggregateRegionObserver|805306366|', coprocessor$4 => '|org.apache.p
hoenix.coprocessor.ServerCachingEndpointImpl|805306366|', coprocessor$5 => '|org.apache.phoenix.hbase.index.Ind
exer|805306366|index.builder=org.apache.phoenix.index.PhoenixIndexBuilder,org.apache.hadoop.hbase.index.codec.c
lass=org.apache.phoenix.index.PhoenixIndexCodec'}                                                              
COLUMN FAMILIES DESCRIPTION                                                                                    
{NAME => '0', VERSIONS => '1', EVICT_BLOCKS_ON_CLOSE => 'false', NEW_VERSION_BEHAVIOR => 'false', KEEP_DELETED_
CELLS => 'FALSE', CACHE_DATA_ON_WRITE => 'false', DATA_BLOCK_ENCODING => 'FAST_DIFF', TTL => 'FOREVER', MIN_VER
SIONS => '0', REPLICATION_SCOPE => '0', BLOOMFILTER => 'NONE', CACHE_INDEX_ON_WRITE => 'false', IN_MEMORY => 'f
alse', CACHE_BLOOMS_ON_WRITE => 'false', PREFETCH_BLOCKS_ON_OPEN => 'false', COMPRESSION => 'NONE', BLOCKCACHE 
=> 'true', BLOCKSIZE => '65536'}                                                                               
1 row(s)
Took 0.0354 seconds  
```

>Note: see something interesting here? What about the coprocessors that are attached to the table?

Now lets use SQL to upsert data:

```sql
0: jdbc:phoenix:localhost> upsert into driver values (1,'John','Doe');
1 row affected (0.044 seconds)
```

>Note: it is still HBase so `upsert` instead of `insert`


And now select the table:

```sql
0: jdbc:phoenix:localhost> select * from driver;
+-----+------------+-----------+
| ID  | FIRSTNAME  | LASTNAME  |
+-----+------------+-----------+
| 1   | John       | Doe       |
+-----+------------+-----------+
1 row selected (0.015 seconds)
```

Now scan is HBase:

```sql
hbase(main):023:0> scan 'DRIVER'
ROW                          COLUMN+CELL                                                                       
 \x80\x00\x00\x01            column=0:\x00\x00\x00\x00, timestamp=1535808055914, value=x                       
 \x80\x00\x00\x01            column=0:\x80\x0B, timestamp=1535808055914, value=John                            
 \x80\x00\x00\x01            column=0:\x80\x0C, timestamp=1535808055914, value=Doe                             
1 row(s)
Took 0.0101 seconds   
```

>Note: what is different about the data now?

Now upsert several more rows using Phoenix SQL and repeat the scan.

Make sense?

If you want to exit, do a `!q`:

```sql
	0: jdbc:phoenix:localhost> !q
	Closing: org.apache.phoenix.jdbc.PhoenixConnection
	[centos@ip-10-0-0-237 bin]$ 
```

### Results

Congratulations! We went through the introduction of Phoenix and how to use it with HBase, and created data from both directions.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>