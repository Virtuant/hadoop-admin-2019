## Indexes and Secondary Indexes in Phoenix/HBase

**Objective**: Use Phoenix to create indexes on HBase and see the performance.

**HBase tables**:     `iot_data`

**Data Source**: `iot_data.csv`

In this exercise you will use the Phoenix to create and use indexes. Phoenix secondary indexes are often misunderstood. Those coming from the relational world mistakenly apply the same principles to Apache Phoenix.

What are Apache Phoenix secondary indexes? Secondary indexes are an orthogonal way to access data from its primary access path. Orthogonal is key here. Think of this as an intersection. Personally I would argue this is different then RDBMS as RDBMS adheres to relational theory. HBase/Phoenix does not. So start training your mind to think of intersections when it comes to secondary indexes.

#### Global Index

Global indexes are used for read heavy use cases. Global indexes are not co-located (region server) with the primary table. Therefore with global indexes are dispersing the read load by have the main and secondary index table on different region servers and serving different set of access patterns. Think of it as load balancing.

#### Functional Indexes

Functional indexes (available in 4.3 and above) allow you to create an index not just on columns, but on an arbitrary expressions. Then when a query uses that expression, the index may be used to retrieve the results instead of the data table. For example, you could create an index on `UPPER(FIRST_NAME||‘ ’||LAST_NAME)` to allow you to do case insensitive searches on the combined first name and last name of a person.

#### Covered index

A covered index is a way to bundle data based on alternative access path. If the index can "cover" all fields in your select statement then only the index will be hit during the query.

#### Local index

Local indexes are used for write heavy use cases. Why? Local indexes are co-located (Region server) with the primary table. Unlike global indexes, local indexes will use an index even when all columns referenced in the query are not contained in the index. This is done by default for local indexes because we know that the table and index data co-reside on the same region server thus ensuring the lookup is local.

Let's use a couple of these to see what happens.

----
### Sample Table

So let's populate our driver table:


```sql
0: jdbc:phoenix:localhost> upsert into s2.driver values (1,'John','Doe');
1 row affected (0.006 seconds)
```

And in HBase:

```sql
hbase(main):025:0> scan 'S2.DRIVER'
ROW                          COLUMN+CELL                                                                       
 \x80\x00\x00\x01            column=0:\x00\x00\x00\x00, timestamp=1535812691474, value=x                       
 \x80\x00\x00\x01            column=0:\x80\x0B, timestamp=1535812691474, value=John                            
 \x80\x00\x00\x01            column=0:\x80\x0C, timestamp=1535812691474, value=Doe                             
1 row(s)
Took 0.0098 seconds   
```

Looks good now a few more rows:

```sql
0: jdbc:phoenix:localhost> upsert into s2.driver values (2,'Mary','Jones');
0: jdbc:phoenix:localhost> upsert into s2.driver values (3,'Bill','Graham');
0: jdbc:phoenix:localhost> upsert into s2.driver values (4,'Ravi','Hejazi');
```

And again, a scan show the rows:

```sql
hbase(main):026:0> scan 'S2.DRIVER'
ROW                          COLUMN+CELL                                                                       
 \x80\x00\x00\x01            column=0:\x00\x00\x00\x00, timestamp=1535812691474, value=x                       
 \x80\x00\x00\x01            column=0:\x80\x0B, timestamp=1535812691474, value=John                            
 \x80\x00\x00\x01            column=0:\x80\x0C, timestamp=1535812691474, value=Doe                             
 \x80\x00\x00\x02            column=0:\x00\x00\x00\x00, timestamp=1535820567885, value=x                       
 \x80\x00\x00\x02            column=0:\x80\x0B, timestamp=1535820567885, value=Mary     
```

So back in Phoenix, a select from this hbase table should be very fast:

```sql
0: jdbc:phoenix:localhost> select * from s2.driver where id = 3;
+-----+--------+---------+
| ID  | FNAME  |  LNAME  |
+-----+--------+---------+
| 3   | Bill   | Graham  |
+-----+--------+---------+
1 row selected (0.014 seconds)
0: jdbc:phoenix:localhost> 
```

>Note: why is that true?

### A Large Table

OK, so what about bigger tables? Let's try it out:

```sql
hbase(main):024:0> create 'iot_data','para'
Created table iot_data
```

Now create the table in Phoenix:

```sql
0: jdbc:phoenix:localhost> CREATE table "iot_data" ( ROWKEY VARCHAR PRIMARY KEY, "para"."deviceId" VARCHAR, "para"."deviceParameter" VARCHAR, "para"."deviceValue" varchar, "para"."dateTime" date) ;
No rows affected (5.753 seconds)
```

Now we'll use ImportTsv to load the table (make sure the data is in HDFS first):

```console
 hbase org.apache.hadoop.hbase.mapreduce.ImportTsv 
  -Dimporttsv.separator=, 
  -Dimporttsv.columns="HBASE_ROW_KEY,para:deviceParameter,para:deviceValue,para:deviceId,para:dateTime" 
  iot_data 
  hdfs://localhost:/tmp/iot_data.csv
```

>Note: easier to do:

```console
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=, -Dimporttsv.columns="HBASE_ROW_KEY,para:deviceParameter,para:deviceValue,para:deviceId,para:dateTime" iot_data hdfs://localhost:/tmp/iot_data.csv
```

You can now scan the table:

```sql
scan 'iot_data', {'LIMIT' => 10}
```

And select a few rows in Phoenix:

```sql
0: jdbc:phoenix:localhost> select * from "iot_data" limit 10;
+-------------------------------------+-----------+------------------+--------------+-------------------------+
|               ROWKEY                | deviceId  | deviceParameter  | deviceValue  |           dateTime      |
+-------------------------------------+-----------+------------------+--------------+-------------------------+
| ObjectId(5a81b5395882b86112555f70)  | SBS05     | Temperature      | 27           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b5395882b86112555f71)  | SBS05     | Humidity         | 59           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53a5882b86112555f72)  | SBS04     | Sound            | 130          | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53b5882b86112555f73)  | SBS05     | Humidity         | 75           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53b5882b86112555f74)  | SBS02     | Temperature      | 33           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53c5882b86112555f75)  | SBS03     | Sound            | 102          | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53c5882b86112555f76)  | SBS02     | Temperature      | 18           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53c5882b86112555f77)  | SBS05     | Flow             | 64           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53d5882b86112555f78)  | SBS04     | Temperature      | 28           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b53d5882b86112555f79)  | SBS03     | Humidity         | 69           | 177670840-07-21 15:49:5 |
+-------------------------------------+-----------+------------------+--------------+-------------------------+
10 rows selected (0.043 seconds)
```

### Create an Index on a Large Table

Do a simple select on the table:

```sql
0: jdbc:phoenix:localhost> select * from "iot_data" where "deviceId" = 'SBS05';
+-------------------------------------+-----------+------------------+--------------+-------------------------+
|               ROWKEY                | deviceId  | deviceParameter  | deviceValue  |           dateTime      |
+-------------------------------------+-----------+------------------+--------------+-------------------------+
| ObjectId(5a81b5395882b86112555f70)  | SBS05     | Temperature      | 27           | 177670840-07-21 15:49:5 |
| ObjectId(5a81b5395882b86112555f71)  | SBS05     | Humidity         | 59           | 177670840-07-21 15:49:5 |
...
| ObjectId(5a9123ba5882b84ceb10c311)  | SBS05     | Temperature      | 33           | 177670840-07-21 15:49:5 |
| ObjectId(5a9123bd5882b84ceb10c315)  | SBS05     | Temperature      | 28           | 177670840-07-21 15:49:5 |
| ObjectId(5a9123be5882b84ceb10c317)  | SBS05     | Sound            | 105          | 177670840-07-21 15:49:5 |
+-------------------------------------+-----------+------------------+--------------+-------------------------+
85,394 rows selected (48.017 seconds)
```

So now create an index on the table:

```sql
0: jdbc:phoenix:localhost> create index "device_index1" on "iot_data" ("para"."deviceId");
426,879 rows affected (14.714 seconds)
```

```sql
select * from "iot_data" where "deviceId" = 'SBS05' AND "deviceValue" = '101'
+-------------------------------------+-----------+------------------+--------------+-------------------------+
|               ROWKEY                | deviceId  | deviceParameter  | deviceValue  |           dateTime      |
+-------------------------------------+-----------+------------------+--------------+-------------------------+
| ObjectId(5a81b5395882b86112555f70)  | SBS05     | Temperature      | 101          | 177670840-07-21 15:49:5 |
| ObjectId(5a81b5395882b86112555f71)  | SBS05     | Humidity         | 101          | 177670840-07-21 15:49:5 |
...
| ObjectId(5a911cf15882b84ceb10b5ab)  | SBS05     | Sound            | 101          | 177670840-07-21 15:49:5 |
| ObjectId(5a91229f5882b84ceb10c0e2)  | SBS05     | Sound            | 101          | 177670840-07-21 15:49:5 |
+-------------------------------------+-----------+------------------+--------------+-------------------------+
637 rows selected (2.509 seconds)
```

Do a list of tables in Phoenix.

```sql
0: jdbc:phoenix:localhost> create index "deviceIdcover" on "iot_data" ("para"."deviceId") include ("para"."deviceParameter");
426,879 rows affected (14.428 seconds)
```

Now do a select on the cover index:

```sql
0: jdbc:phoenix:localhost> select * from "deviceIdcover" limit 10;
+----------------+-------------------------------------+-----------------------+
| para:deviceId  |               :ROWKEY               | para:deviceParameter  |
+----------------+-------------------------------------+-----------------------+
| SBS01          | ObjectId(5a81b5435882b86112555f86)  | Flow                  |
| SBS01          | ObjectId(5a81b5445882b86112555f88)  | Flow                  |
| SBS01          | ObjectId(5a81b54b5882b86112555f96)  | Temperature           |
| SBS01          | ObjectId(5a81b54b5882b86112555f97)  | Temperature           |
| SBS01          | ObjectId(5a81b5515882b86112555fa5)  | Temperature           |
| SBS01          | ObjectId(5a81b5575882b86112555fb3)  | Flow                  |
| SBS01          | ObjectId(5a81b55f5882b86112555fce)  | Flow                  |
| SBS01          | ObjectId(5a81b5615882b86112555fd2)  | Sound                 |
| SBS01          | ObjectId(5a81b5615882b86112555fd3)  | Temperature           |
| SBS01          | ObjectId(5a81b5685882b86112555fe0)  | Temperature           |
+----------------+-------------------------------------+-----------------------+
10 rows selected (0.012 seconds)
```

Explain the query:

```sql
0: jdbc:phoenix:localhost> EXPLAIN select * from "iot_data";
+--------------------------------------------------------------------+-----------------+----------------+-----+
|                                PLAN                                | EST_BYTES_READ  | EST_ROWS_READ  | EST |
+--------------------------------------------------------------------+-----------------+----------------+-----+
| CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER iot_data  | null            | null           | nul |
+--------------------------------------------------------------------+-----------------+----------------+-----+
1 row selected (0.022 seconds)
```

And explain the other select:

```sql
0: jdbc:phoenix:localhost> EXPLAIN select * from "iot_data" where "deviceId" = 'SBS05';
+--------------------------------------------------------------------+-----------------+----------------+-----+
|                                PLAN                                | EST_BYTES_READ  | EST_ROWS_READ  | EST |
+--------------------------------------------------------------------+-----------------+----------------+-----+
| CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER iot_data  | null            | null           | nul |
|     SERVER FILTER BY para."deviceId" = 'SBS05'                     | null            | null           | nul |
+--------------------------------------------------------------------+-----------------+----------------+-----+
2 rows selected (0.015 seconds)
0: jdbc:phoenix:localhost> 
```

And explain another select:

```sql
0: jdbc:phoenix:localhost> EXPLAIN select * from "iot_data" where "deviceParameter" = 'Sound';
+--------------------------------------------------------------------+-----------------+----------------+-----+
|                                PLAN                                | EST_BYTES_READ  | EST_ROWS_READ  | EST |
+--------------------------------------------------------------------+-----------------+----------------+-----+
| CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER iot_data  | null            | null           | nul |
|     SERVER FILTER BY para."deviceParameter" = 'Sound'              | null            | null           | nul |
+--------------------------------------------------------------------+-----------------+----------------+-----+
2 rows selected (0.016 seconds)
```

And this select:

```sql
0: jdbc:phoenix:localhost> EXPLAIN select * from "iot_data" where "deviceId" = 'SBS05' and "deviceParameter" = 'Sound';
+----------------------------------------------------------------------------------------+-----------------+--+
|                                          PLAN                                          | EST_BYTES_READ  |  |
+----------------------------------------------------------------------------------------+-----------------+--+
| CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER iot_data                      | null            |  |
|     SERVER FILTER BY (para."deviceId" = 'SBS05' AND para."deviceParameter" = 'Sound')  | null            |  |
+----------------------------------------------------------------------------------------+-----------------+--+
2 rows selected (0.015 seconds)
```

And do the select:

```sql
0: jdbc:phoenix:localhost> select * from "iot_data" where "deviceId" = 'SBS05' and "deviceParameter" = 'Sound'
```

### Results

So now you see the use of indexes on a table. Pretty interesting huh? 


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>