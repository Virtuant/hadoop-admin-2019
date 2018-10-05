# Possible replacement info:

https://hortonworks.com/hadoop-tutorial/introduction-apache-hbase-concepts-apache-phoenix-new-backup-restore-utility-hbase/

https://community.hortonworks.com/questions/82129/how-to-fetch-single-column-from-hbase-table.html

https://community.hortonworks.com/questions/72251/creating-two-column-familes-with-one-sqoop-command.html

https://community.hortonworks.com/questions/66767/hbase-complete-list-of-columns-in-column-family.html

https://community.hortonworks.com/questions/55530/insert-into-a-hbase-table-with-multiple-column-fam.html


## Lab: Column Families

HBase is a distributed column-oriented database built on top of the Hadoop file system. It is an open-source project and is horizontally scalable. HBase is a data model that is similar to Google’s big table designed to provide quick random access to huge amounts of unstructured data. It leverages the fault tolerance provided by the Hadoop File System (HDFS).

The components of HBase data model consist of tables, rows, column families, columns, cells and versions. Tables are like logical collection of rows stored in separate partitions. A row is one instance of data in a table and is identified by a rowkey. Data in a row are grouped together as Column Families. Each Column Family has one or more Columns and these Columns in a family are stored together. Column Families form the basic unit of physical storage, hence it’s important that proper care be taken when designing Column Families in table. A Column is identified by a Column Qualifier that consists of the Column Family name concatenated with the Column name using a colon. A Cell stores data and is essentially a unique combination of rowkey, Column Family and the Column (Column Qualifier). The data stored in a cell is versioned and versions of data are identified by the timestamp.


## Objective:
Create a table with different column families and explore the physical layout of the table in hdfs.

### File locations:

### Successful outcome:
You will:
create a table with different column families and explore the physical layout of the table in hdfs

## 1. START HBASE

### 1.1 VIEW THE HBASE SERVICES PAGE
In order to start/stop HBase service, you must log into Ambari as an administrator. The default account (maria_dev) will not allow you to do this. Please follow these step to setup password for admin account.
First SSH into the Hortonworks Sandbox with the command:

# Image goes here:
# Another image goes here:

If do do not have ssh client, you can also access the shell via http://localhost:4200/
Now run the following command to reset the password for user admin:

# Two images go here one on top of the other 

Now navigate to Ambari on 127.0.0.1:8080 on the browser and give your credentials

From the Dashboard page of Ambari, click on HBase from the list of installed services.

# Image goes here:

1.2 START HBASE SERVICE
From the HBase page, click on Service Actions -> 

# Image goes here

Check the box and click on Confirm Start:

# Image goes here:

Check the box to turn off the Maintenance Mode as it suppresses alerts, warnings and status change indicators generated for the object.
Wait for HBase to start (It may take a few minutes to turn green)

# Image goes here: 

## 2. ENTER HBASE SHELL
HBase comes with an interactive shell from where you can communicate with HBase components and perform operations on them.

First SSH into the Hortonworks Sandbox with the command:

Switch the user to hbase.

# 2 Images goes here:

Type hbase shell and you will see the following screen:

# Image goes here:

To exit the interactive shell, type exit or use <ctrl+c>. But wait, it is time to explore more features of the shell.

## 3. DATA DEFINITION LANGUAGE COMMANDS IN HBASE
These are the commands that operate on tables in HBase.

## 3.1 CREATE
The syntax to create a table in HBase is create '<table_name>','<column_family_name>'. Let’s create a table called ‘driver_dangerous_event’ with a column family of name events. Run the following command:

# Two images go here:

## 3.2 LIST
Let’s check the table we’ve just created, type the following command in the HBase shell

# image goes here:

## 4. DATA MANIPULATION COMMANDS IN HBASE
Let’s import some data into the table. We’ll use a sample dataset that tracks driving record of a logistics company.

Open a new terminal and ssh into the Sandbox. Download the data.csv file and let’s copy the file in HDFS,

## Now execute the LoadTsv from hbase user statement as following:

# image goes here: 

Now let’s check whether the data got imported in the table driver_dangerous_events or not. Go back to the hbase shell.

## 4.1 SCAN
The scan command is used to view the data in the HBase table. Type the following command:
scan 'driver_dangerous_event'

You will see all the data present in the table with row keys and the different values for different columns in a column family.

# Image goes here:

## 4.2 PUT
Using put command, you can insert rows in a HBase table. The syntax of put command is as follows:
put '<table_name>','row1','<column_family:column_name>','value'

Copy following lines to put the data in the table.

# 2 Image goes here:

Now let’s view a data from scan command.

# 2 images go here:

You can also update an existing cell value using the put command. The syntax for replacing is same as inserting a new value.

So let’s update a route name value of row key 4, from 'Santa Clara to San Diego' to 'Santa Clara to Los Angeles'. Type the following command in HBase shell:

hbase>put 'driver_dangerous_event','4','events:routeName','Santa Clara to Los Angeles'

# Image goes here: 

Now scan the table to see the updated data:

# 2 images goes here: 

## 4.3 GET
It is used to read the data from HBase table. It gives a single row of data at a time. Syntax for get command is:
get '<table_name>','<row_number>'

Try typing get 'driver_dangerous_event','1' in the shell. You will see the all the column families (in our case, there is only 1 column family) along with all the columns in the row.

# Image goes here:

You can also read a specific column from get command. The syntax is as follows:
get 'table_name', 'row_number', {COLUMN ⇒ 'column_family:column-name '}

Type the following statement to get the details from the row 1 and the driverName of column family events.

hbase>get 'driver_dangerous_event','1',{COLUMN => 'events:driverName'}

# Image goes here: 

If you want to view the data from two columns, just add it to the {COLUMN =>…} section. Run the following command to get the details from row key 1 and the driverName and routeId of column family events:

# Image goes here:

### SUMMARY
In this tutorial, we learned about the basic concepts of Apache HBase and different types of data definition and data manipulation commands that are available in HBase shell including column families. 




### Lab Steps

Complete the setup guide

You are to create a table with different column families and explore the physical layout of the table in hdfs

----
1.	In the HBase Shell, create a table called cf with two column families, column family a will store a single version of each cell, column family b will store up to 3 versions of each cell

    ```console
    hbase> create 'cf',{NAME=>'a', VERSIONS =>1},{NAME=>'b', VERSIONS=>2}
    ```
    
    > Note: this is important

1.	Insert some cells into each column family:

    ```console
    hbase> put 'cf','1','a:name','yourname'
    0 row(s) in 0.0200 seconds
    ```
	
3.	Retrieve all cells for rowkey 1:

    ```console
    hbase> get 'cf',1
    COLUMN	CELL
    a:name	timestamp=1396035080378, value=yourname
    ```
    
5.	Put salary into column family 'b':

    ```console
    hbase> put 'cf','1','b:salary','50,000'
    ```
 
6.	Retrieve all cells for rowkey 1:

    ```console
    hbase> get 'cf',1
    COLUMN	CELL
    a:name	timestamp=1396035080378, value=yourname
    CELL
    b:salary	timestamp=1396035310760, value=yourname
    ```
    
7.	Enter a new salary for rowkey '1' into column family b:

    ```console
    hbase> put 'cf','1','b:salary','77,000'
    ```
    
8.	Retrieve multiple versions of the cells for rowkey 1:

    ```console
    hbase> get 'cf',1,{COLUMN => ['a','b'],VERSIONS=>2} 
    COLUMN      CELL
    a:name	    timestamp=1396035080378, value=yourname
    b:salary	timestamp=1396035310760, value=77,000
    b:salary	timestamp=1396035206632, value=50,000
    ```
    
9.	Flush the table. The rows just inserted are in a memory cache. Flush them to hdfs:

    ```console
    hbase> flush 'cf'
    0 row(s) in 0.1460 seconds
    ```
    
1.	Find the data directories for each column family. Quit the HBase shell and run this hdfs command:

    ```console
    $ hdfs dfs -ls /apps/hbase/data/data/default/cf 
    Found 3 items
    drwxr-xr-x	- hbase hdfs  0 2014-03-28 12:31  /apps/hbase/data/data/default/cf/.tabledesc
    drwxr-xr-x	- hbase hdfs  0 2014-03-28 12:31  /apps/hbase/data/data/default/cf/.tmp
    drwxr-xr-x	- hbase hdfs  0 2014-03-28 12:43  /apps/hbase/data/data/default/cf/98491258b8d8b1dd5e7d84478a6f3290
    ```
    
    This shows that the data for table cf is stored in hdfs under `/apps/hbase/data/data/default/cf`

11.	The directory with the hex number for a name is the version number for this table; this number will be different. Use the following command to see the content:

    ```console
    $ hdfs dfs -ls -r /apps/hbase/data/data/default/cf 
    Found 1 items
    -rw-r--r--	3 hbase hdfs  569 2014-03-28 12:31 /apps/hbase/data/data/default/cf/.tabledesc/.tableinfo.0000000001 
    Found 4 items
    -rwxr-xr-x	3 hbase hdfs   35 2014-03-28 12:31 /apps/hbase/data/data/default/cf/98491258b8d8b1dd5e7d84478a6f3290/.regioninfo 
    drwxr-xr-x	- hbase hdfs	0 2014-03-28 12:43 /apps/hbase/data/data/default/cf/98491258b8d8b1dd5e7d84478a6f3290/.tmp 
    drwxr-xr-x	- hbase hdfs	0 2014-03-28 12:43 /apps/hbase/data/data/default/cf/98491258b8d8b1dd5e7d84478a6f3290/a 
    drwxr-xr-x	- hbase hdfs	0 2014-03-28 12:43 /apps/hbase/data/data/default/cf/98491258b8d8b1dd5e7d84478a6f3290/b
    ```
    
    There is a directory a for data in column family a, and a directory b for data in column family b.

### RESULT:

You have now created an HBase table with multiple column families.
