## Importing Data from Hive to HBase

Suppose we have data in Hive table. We want the same data into HBase table. So, our requirement is to migrate the data from Hive to HBase table.

    Hive – Source table
    HBase – Target Table

----

We cannot load data directly into HBase table from the hive. In order to achieve the requirement, we have to go through the following steps.

### Create Hive Table

We are creating this hive table as a source. This table data, we want in HBase table.

    CREATE TABLE iot_data(
      id string,
      parameter string,
      device_value int,
      deviceid string,
      datetime string)
    ROW FORMAT DELIMITED
      FIELDS TERMINATED BY '\t'
      LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;

### Load Data into Hive

Loading the data from the local path. In my case, the local path is /root/bdp/hbase/data/emp_data.csv.

    LOAD DATA LOCAL INPATH '[file path]' INTO TABLE iot_data;

### Create HBase-Hive Mapping table

In this step, we are creating another hive table which actually points to an HBase table:

    CREATE TABLE iot_incoming
      ( 
          id string,
          parameter string,
          device_value int,
          deviceid string,
          datetime string
      )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:id,cf:parameter,cf:device_value,cf:deviceid,cf:datetime")
    TBLPROPERTIES ("hbase.table.name" = "iot_incoming");

Here, we are specifying HBaseStorageHandler in `STORED BY` option. Also, mapping the hive column with HBase column family using `SERDEPROPERTIES`. It will create an HBase table named iot_incoming which will point to this hive table.

>Note: `id` represents the first column of the hive table. So it will become the key.

In case, you are already having an HBase table and want to load data into existing HBase table, then you have to use `EXTERNAL` in your above hive DDL. Otherwise, you will get an error:

```console
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:MetaException(message:Table employee_hbase_2 already exists within HBase; use CREATE EXTERNAL TABLE instead to register it in Hive.)
```

    CREATE EXTERNAL TABLE iot_incoming2
      ( 
          id string,
          parameter string,
          device_value int,
          deviceid string,
          datetime string
      )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:id,cf:parameter,cf:device_value,cf:deviceid,cf:datetime")
    TBLPROPERTIES ("hbase.table.name" = "iot_incoming2");

### Load Data into HBase from Hive

In this step, we are going to migrate hive table data to HBase. That means we will load the hive data to hive table.

    INSERT INTO TABLE iot_incoming SELECT * FROM iot_data;

We have loaded data into iot_incoming table which is pointing to HBase table iot_data.


------------------ cut here ?
The reason is, HBase table will ignore that record.(
      id string,
      parameter string,
      device_value int,
      deviceid string,
      datetime, string)
    ROW FORMAT DELIMITED
      FIELDS TERMINATED BY '\t'
      LINES TERMINATED BY '\n'
    STORED AS TEXTFILE;

### Load Data into Hive

Loading the data from the local path. In my case, the local path is `~/data/hbase/data/iot_data.csv`.
 
    LOAD DATA LOCAL INPATH '[file path]' INTO TABLE hive_table;


### Create HBase-Hive Mapping table

In this step, we are creating another hive table which actually points to an HBase table:
 

    CREATE TABLE hbase_table_employee 
      ( 
         empno       INT, 
         ename       STRING, 
         designation STRING, 
         manager     INT, 
         hire_date   STRING, 
         sal         INT, 
         deptno      INT
      )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:ename,cf:designation,cf:manager,cf:hire_date,cf:sal,cf:deptno")
    TBLPROPERTIES ("hbase.table.name" = "employee_hbase");

Here, we are specifying HBaseStorageHandler in Stored By option. Also, mapping the hive column with HBase column family using SERDEPROPERTIES. It will create an HBase table named employee_hbase which will point to this hive table.

>Note: Key represents the first column of the hive table. So Id will become the key.

In case, you are already having an HBase table and want to load data into existing HBase table, then you have to use EXTERNAL in your above hive DDL. Otherwise, you will get an error:

FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.DDLTask. MetaException(message:MetaException(message:Table employee_hbase_2 already exists within HBase; use CREATE EXTERNAL TABLE instead to register it in Hive.)
 
 

    CREATE EXTERNAL TABLE hbase_table_employee_2
      ( 
         empno       INT, 
         ename       STRING, 
         designation STRING, 
         manager     INT, 
         hire_date   STRING, 
         sal         INT, 
         deptno      INT
      )
    STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
    WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf:ename,cf:designation,cf:manager,cf:hire_date,cf:sal,cf:deptno")
    TBLPROPERTIES ("hbase.table.name" = "employee_hbase_2");

### Load data into HBase from Hive

In this step, we are going to migrate hive table data to HBase. That means we will load the hive (created in step 1) data to hive table created in step 3.
 
 

    INSERT INTO TABLE hbase_table_employee SELECT * FROM hive_table;

We have loaded data into hbase_table_employee table which is pointing to HBase table employee_hbase.
Step 5: Scan HBase Table

Let’s check the data in HBase table:
 
    hbase(main):008:0> scan 'employee_hbase'
    ROW                             COLUMN+CELL
     7369                           column=cf:deptno, timestamp=1514476352028, value=20
     7369                           column=cf:designation, timestamp=1514476352028, value=CLERK
     7369                           column=cf:ename, timestamp=1514476352028, value=SMITH
     7369                           column=cf:hire_date, timestamp=1514476352028, value=12/17/1980
     7369                           column=cf:manager, timestamp=1514476352028, value=7902
     7369                           column=cf:sal, timestamp=1514476352028, value=800
     7499                           column=cf:deptno, timestamp=1514476352028, value=30
     7499                           column=cf:designation, timestamp=1514476352028, value=SALESMAN
     7499                           column=cf:ename, timestamp=1514476352028, value=ALLEN
     7499                           column=cf:hire_date, timestamp=1514476352028, value=2/20/1981
     7499                           column=cf:manager, timestamp=1514476352028, value=7698
     7499                           column=cf:sal, timestamp=1514476352028, value=1600
     7521                           column=cf:deptno, timestamp=1514476352028, value=30
     7521                           column=cf:designation, timestamp=1514476352028, value=SALESMAN
     7521                           column=cf:ename, timestamp=1514476352028, value=WARD
     7521                           column=cf:hire_date, timestamp=1514476352028, value=2/22/1981
     7521                           column=cf:manager, timestamp=1514476352028, value=7698
     7521                           column=cf:sal, timestamp=1514476352028, value=1250
     7566                           column=cf:deptno, timestamp=1514476352028, value=20
     7566                           column=cf:designation, timestamp=1514476352028, value=MANAGER
     7566                           column=cf:ename, timestamp=1514476352028, value=TURNER
     7566                           column=cf:hire_date, timestamp=1514476352028, value=4/2/1981
     7566                           column=cf:manager, timestamp=1514476352028, value=7839
     7566                           column=cf:sal, timestamp=1514476352028, value=2975
     7654                           column=cf:deptno, timestamp=1514476352028, value=30
     7654                           column=cf:designation, timestamp=1514476352028, value=SALESMAN
     7654                           column=cf:ename, timestamp=1514476352028, value=MARTIN
     7654                           column=cf:hire_date, timestamp=1514476352028, value=9/28/1981
     7654                           column=cf:manager, timestamp=1514476352028, value=7698
     7654                           column=cf:sal, timestamp=1514476352028, value=1250
     7698                           column=cf:deptno, timestamp=1514476352028, value=30
     7698                           column=cf:designation, timestamp=1514476352028, value=MANAGER
     7698                           column=cf:ename, timestamp=1514476352028, value=MILLER
     7698                           column=cf:hire_date, timestamp=1514476352028, value=5/1/1981
     7698                           column=cf:manager, timestamp=1514476352028, value=7839
     7698                           column=cf:sal, timestamp=1514476352028, value=2850
     7782                           column=cf:deptno, timestamp=1514476352028, value=10
     7782                           column=cf:designation, timestamp=1514476352028, value=MANAGER
     7782                           column=cf:ename, timestamp=1514476352028, value=CLARK
     7782                           column=cf:hire_date, timestamp=1514476352028, value=6/9/1981
     7782                           column=cf:manager, timestamp=1514476352028, value=7839
     7782                           column=cf:sal, timestamp=1514476352028, value=2450
     7788                           column=cf:deptno, timestamp=1514476352028, value=20
     7788                           column=cf:designation, timestamp=1514476352028, value=ANALYST
     7788                           column=cf:ename, timestamp=1514476352028, value=SCOTT
     7788                           column=cf:hire_date, timestamp=1514476352028, value=12/9/1982
     7788                           column=cf:manager, timestamp=1514476352028, value=7566
     7788                           column=cf:sal, timestamp=1514476352028, value=3000
     7839                           column=cf:deptno, timestamp=1514476352028, value=10
     7839                           column=cf:designation, timestamp=1514476352028, value=PRESIDENT
     7839                           column=cf:ename, timestamp=1514476352028, value=KING
     7839                           column=cf:hire_date, timestamp=1514476352028, value=11/17/1981
     7839                           column=cf:sal, timestamp=1514476352028, value=5000
     7844                           column=cf:deptno, timestamp=1514476352028, value=30
     7844                           column=cf:designation, timestamp=1514476352028, value=SALESMAN
     7844                           column=cf:ename, timestamp=1514476352028, value=TURNER
     7844                           column=cf:hire_date, timestamp=1514476352028, value=9/8/1981
     7844                           column=cf:manager, timestamp=1514476352028, value=7698
     7844                           column=cf:sal, timestamp=1514476352028, value=1500
     7876                           column=cf:deptno, timestamp=1514476352028, value=20
     7876                           column=cf:designation, timestamp=1514476352028, value=CLERK
     7876                           column=cf:ename, timestamp=1514476352028, value=ADAMS
     7876                           column=cf:hire_date, timestamp=1514476352028, value=1/12/1983
     7876                           column=cf:manager, timestamp=1514476352028, value=7788
     7876                           column=cf:sal, timestamp=1514476352028, value=1100
     7900                           column=cf:deptno, timestamp=1514476352028, value=30
     7900                           column=cf:designation, timestamp=1514476352028, value=CLERK
     7900                           column=cf:ename, timestamp=1514476352028, value=JAMES
     7900                           column=cf:hire_date, timestamp=1514476352028, value=12/3/1981
     7900                           column=cf:manager, timestamp=1514476352028, value=7698
     7900                           column=cf:sal, timestamp=1514476352028, value=950
     7902                           column=cf:deptno, timestamp=1514476352028, value=20
     7902                           column=cf:designation, timestamp=1514476352028, value=ANALYST
     7902                           column=cf:ename, timestamp=1514476352028, value=FORD
     7902                           column=cf:hire_date, timestamp=1514476352028, value=12/3/1981
     7902                           column=cf:manager, timestamp=1514476352028, value=7566
     7902                           column=cf:sal, timestamp=1514476352028, value=3000
     7934                           column=cf:deptno, timestamp=1514476352028, value=10
     7934                           column=cf:designation, timestamp=1514476352028, value=CLERK
     7934                           column=cf:ename, timestamp=1514476352028, value=MILLER
     7934                           column=cf:hire_date, timestamp=1514476352028, value=1/23/1982
     7934                           column=cf:manager, timestamp=1514476352028, value=7782
     7934                           column=cf:sal, timestamp=1514476352028, value=1300
    14 row(s) in 1.7580 seconds

### Summary

In this post, we have created a hive to hbase mapping table in order to migrate data from hive to hbase. There is an HBase table on top of our Hive table. If your hive table contains a record which has NULL values for all the columns, in that case, hive and hbase records count would differ. The reason is, HBase table will ignore that record.
