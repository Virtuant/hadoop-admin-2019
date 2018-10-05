# links for information on this page: 

https://hortonworks.com/hadoop-tutorial/introduction-apache-hbase-concepts-apache-phoenix-new-backup-restore-utility-hbase/

https://hortonworks.com/apache/phoenix/

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_command-line-installation/content/configuring-hbase-for-phoenix.html

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_data-access/content/ch_using-phoenix.html

https://community.hortonworks.com/questions/21863/how-to-create-phoenix-table-using-existing-hbase-t.html

## Objective:
Learn how to take your SQL query, compile it into a series of HBase scans, and orchestrate the running of those scans to produce regular JDBC result sets.

### File locations:

### Successful outcome:
You will:
Have direct use of the HBase API, along with coprocessors and custom filters, results in performance on the order of milliseconds for small queries, or seconds for tens of millions of rows.

### Lab Steps

Complete the setup guide
## Lab: Phoenix on HBase

Phoenix takes your SQL query, compiles it into a series of HBase scans, and orchestrates the running of those scans to produce regular JDBC result sets. Direct use of the HBase API, along with coprocessors and custom filters, results in performance on the order of milliseconds for small queries, or seconds for tens of millions of rows.

----

1. Switch the user to hbase:

		# su - hbase

	To exit the interactive shell, type exit or use `<ctrl+c>`.

1. Create a table and list it:

		hbase> create 'driver_dangerous_event','events'
		hbase> list

1. Let’s import some data into the table. We’ll use a sample dataset that tracks driving record of a logistics company. Download the data.csv file and let’s copy the file in HDFS:

		# curl -o ~/data.csv https://raw.githubusercontent.com/hortonworks/data-tutorials/d0468e45ad38b7405570e250a39cf998def5af0f/tutorials/hdp/hdp-2.5/introduction-to-apache-hbase-concepts-apache-phoenix-and-new-backup-restore-utility-in-hbase/assets/data.csv

		# hdfs dfs -copyFromLocal ~/data.csv /tmp

1. Now execute the LoadTsv from hbase user statement as following:

		# su - hbase
		# hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=,  -Dimporttsv.columns="HBASE_ROW_KEY,events:driverId,events:driverName,events:eventTime,events:eventType,events:latitudeColumn,events:longitudeColumn,events:routeId,events:routeName,events:truckId" driver_dangerous_event hdfs://sandbox.hortonworks.com:/tmp/data.csv

	Now let’s check whether the data got imported in the table `driver_dangerous_events` or not. 

1. Go back to the hbase shell, and do a scan:

		hbase> scan 'driver_dangerous_event'

	You will see all the data present in the table with row keys and the different values for different columns in a column family.

1. Using put command, you can insert rows in a HBase table like this:

		hbase> put '<table_name>','row1','<column_family:column_name>','value'

	So copy following lines to put the data in the table:

		put 'driver_dangerous_event','4','events:driverId','78'
		put 'driver_dangerous_event','4','events:driverName','Carl'
		put 'driver_dangerous_event','4','events:eventTime','2016-09-23 03:25:03.567'
		put 'driver_dangerous_event','4','events:eventType','Normal'
		put 'driver_dangerous_event','4','events:latitudeColumn','37.484938'
		put 'driver_dangerous_event','4','events:longitudeColumn','-119.966284'
		put 'driver_dangerous_event','4','events:routeId','845'
		put 'driver_dangerous_event','4','events:routeName','Santa Clara to San Diego'
		put 'driver_dangerous_event','4','events:truckId','637'

1. Now let’s view a data from scan command:

		hbase> scan 'driver_dangerous_event'

	You can also update an existing cell value using the put command. The syntax for replacing is same as inserting a new value.

1. So let’s update a route name value of row key 4, from 'Santa Clara to San Diego' to 'Santa Clara to Los Angeles'. Type the following command in HBase shell:

		hbase>put 'driver_dangerous_event','4','events:routeName','Santa Clara to Los Angeles'

1. Now scan the table to see the updated data:

		hbase> scan 'driver_dangerous_event'

1. Now for a GET to retrive the details from the row 1 and the driverName of column family events:

		hbase> get 'driver_dangerous_event','1',{COLUMN => 'events:driverName'}

	If you want to view the data from two columns, just add it to the {COLUMN =>…} section. Run the following command to get the details from row key 1 and the driverName and routeId of column family events:

		hbase> get 'driver_dangerous_event','1',{COLUMNS => ['events:driverName','events:routeId']}

### Phoenix Shell

To connect to Phoenix, you need to specify the zookeeper quorum and in the sandbox, it is localhost. To launch it, execute the following commands:

		# cd /usr/hdp/current/phoenix-client/bin
		# ./sqlline.py localhost

You can create a Phoenix table/view on a pre-existing HBase table. There is no need to move the data to Phoenix or convert it. Apache Phoenix supports table creation and versioned incremental alterations through DDL commands. The table metadata is stored in an HBase table and versioned. 

You can either create a READ-WRITE table or a READ only view with a condition that the binary representation of the row key and key values must match that of the Phoenix data types. The only addition made to the HBase table is Phoenix coprocessors used for query processing. A table can be created with the same name.

	> NOTE: The DDL used to create the table is case sensitive and if HBase table name is in lowercase, you have to put the name in between double quotes. In HBase, you don’t model the possible KeyValues or the structure of the row key. This is the information you specify in Phoenix and beyond the table and column family.

1. Show help screen:

		localhost> help

1. Create a Phoenix table from existing HBase table by writing a code like this:

		localhost> create table "driver_dangerous_event" ("row" VARCHAR primary key,"events"."driverId" VARCHAR,
		"events"."driverName" VARCHAR,"events"."eventTime" VARCHAR,"events"."eventType" VARCHAR,
		"events"."latitudeColumn" VARCHAR,"events"."longitudeColumn" VARCHAR,
		"events"."routeId" VARCHAR,"events"."routeName" VARCHAR,"events"."truckId" VARCHAR);

1. Describe the table:

		localhost> !describe "driver_dangerous_event"

1. You can view the HBase table data from this Phoenix table:

		localhost> select * from "driver_dangerous_event";

1. If you want to change the view from horizontal to vertical, type the following command in the shell and then try to view the data again:

		!outputformat vertical
		localhost> select * from "driver_dangerous_event";

	If you do not like this view, you can change it back to horizontal view by the following:

		!outputformat horizontal

So with all existing HBase tables, you can query them with SQL now. You can point your Business Intelligence tools and Reporting Tools and other tools which work with SQL and query HBase as if it was another SQL database with the help of Phoenix.

### Inserting Data via Phoenix

You can insert the data using UPSERT command. It inserts if not present and updates otherwise the value in the table. The list of columns is optional and if not present, the values will map to the column in the order they are declared in the schema. 

1. Copy the UPSERT statement given below and then view the newly added row:

		localhost> UPSERT INTO "driver_dangerous_event" values('5','23','Matt','2016-02-29 12:35:21.739','Abnormal','23.385908','-101.384927','249','San Tomas to San Mateo','814');
		localhost> select * from "driver_dangerous_event";

	You will see a newly added row.


### Result

Congratulations! We went through the introduction of Phoenix and how to use it with HBase.
