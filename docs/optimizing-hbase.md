## Lab: Optimizing NOSql in Hive and HBase

----

So create another `users` table. This time we will split the table into 4 regions:

```sql
hbase(main):004:0> create 'users_optimized','f1', SPLITS => ['0250000','0500000','0750000']
Created table users_optimized
Took 2.2568 seconds
=> Hbase::Table - users_optimized
```

>Question: how are the regions split?

Now create the related table:

```sql
hbase(main):004:0> create 'users_name','f1'
Created table users_name
Took 1.1115 seconds
=> Hbase::Table - users_name
```

Let's populate the optimizable table. The incoming file looks like this:

```console
0000001Alexandre Sporer24594 Emmitt Locks, Greenfelderview, MT 481282007-05-0207683017318MT
0000002Dr. Kaylen Leannon DVM75912 Walsh Divide Apt. 025, Port Meghanport, NV 95009-70582004-04-03230.333.9757x28816NV
0000003Novella Fritsch235 Streich Plain Apt. 832, New Arielchester, WY 314571986-03-07652.068.4150WY
0000004Shade Hoeger8250 Muller Port Apt. 962, Stokestown, WI 601531984-07-091-568-851-6573WI
```

So we'll use the hadoop util called [ImportTsv](http://hbase.apache.org/0.94/book/ops_mgt.html#importtsv):

```console
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv 
-Dimporttsv.columns=HBASE_ROW_KEY,f1:name,f1:address,f1:dob,f1:phone,f1:state 
-Dimporttsv.separator=$'\x01' users_optimized data_padded.hive
```

>Note: copyable command:

```console
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns=HBASE_ROW_KEY,f1:name,f1:address,f1:dob,f1:phone,f1:state -Dimporttsv.separator=$'\x01' users_optimized data_padded.hive
```

You should see something like this:

```console
2018-08-31 12:35:32,961 INFO  [main] mapreduce.Job: Job job_1535714493996_0004 running in uber mode : false
2018-08-31 12:35:32,962 INFO  [main] mapreduce.Job:  map 0% reduce 0%
2018-08-31 12:35:44,078 INFO  [main] mapreduce.Job:  map 27% reduce 0%
2018-08-31 12:35:47,088 INFO  [main] mapreduce.Job:  map 48% reduce 0%
2018-08-31 12:35:50,099 INFO  [main] mapreduce.Job:  map 66% reduce 0%
2018-08-31 12:35:53,108 INFO  [main] mapreduce.Job:  map 86% reduce 0%
2018-08-31 12:35:55,125 INFO  [main] mapreduce.Job:  map 100% reduce 0%
2018-08-31 12:35:55,130 INFO  [main] mapreduce.Job: Job job_1535714493996_0004 completed successfully
2018-08-31 12:35:55,227 INFO  [main] mapreduce.Job: Counters: 33
        File System Counters
                FILE: Number of bytes read=0
                FILE: Number of bytes written=270832
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=103157240
                HDFS: Number of bytes written=0
                HDFS: Number of read operations=2
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=0
        Job Counters
                Launched map tasks=1
                Data-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=95855
                Total time spent by all reduces in occupied slots (ms)=0
                Total time spent by all map tasks (ms)=19171
                Total vcore-milliseconds taken by all map tasks=19171
                Total megabyte-milliseconds taken by all map tasks=98155520
        Map-Reduce Framework
                Map input records=1000000
                Map output records=1000000
                Input split bytes=121
                Spilled Records=0
                Failed Shuffles=0
                Merged Map outputs=0
                GC time elapsed (ms)=139
                CPU time spent (ms)=11130
                Physical memory (bytes) snapshot=458571776
                Virtual memory (bytes) snapshot=6446751744
                Total committed heap usage (bytes)=367525888
                Peak Map Physical memory (bytes)=458571776
                Peak Map Virtual memory (bytes)=6457593856
        ImportTsv
                Bad Lines=0
        File Input Format Counters
                Bytes Read=103157119
        File Output Format Counters
                Bytes Written=0
```

Now populate the related table:


```console
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv 
-Dimporttsv.columns=f1:id,HBASE_ROW_KEY,f1:address,f1:dob,f1:phone,f1:state 
-Dimporttsv.separator=$'\x01' users_name data_padded.hive
```

>Note: copyable command:

```console
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.columns=f1:id,HBASE_ROW_KEY,f1:address,f1:dob,f1:phone,f1:state -Dimporttsv.separator=$'\x01' users_name data_padded.hive
```

>Note: do you see anything different about this file?

The `HBASE_ROW_KEY` in this case is pointing to the name, rather than `id`.

### Look at Data

Now go back into the hbase shell, and look at 1 row of the main table:

```sql
hbase(main):004:0> scan 'users_optimized', LIMIT=>1
ROW          COLUMN+CELL
 0000001     column=f1:address, timestamp=1535718910795, value=24594 Emmitt Locks, Greenfelderview, MT 48128
 0000001     column=f1:dob, timestamp=1535718910795, value=2007-05-02
 0000001     column=f1:name, timestamp=1535718910795, value=Alexandre Sporer
 0000001     column=f1:phone, timestamp=1535718910795, value=07683017318
 0000001     column=f1:state, timestamp=1535718910795, value=MT
```

And the related table:

```sql
hbase(main):008:0> scan 'users_name', LIMIT=>1
ROW                COLUMN+CELL
 Aaden Anderson    column=f1:address, timestamp=1535719572328, value=0554 Yost Glen, West Ishmael, UT 77052
 Aaden Anderson    column=f1:dob, timestamp=1535719572328, value=2006-04-28
 Aaden Anderson    column=f1:id, timestamp=1535719572328, value=0867620
 Aaden Anderson    column=f1:phone, timestamp=1535719572328, value=1-227-579-5454
 Aaden Anderson    column=f1:state, timestamp=1535719572328, value=UT
1 row(s)
Took 0.4199 seconds
```

>Note: the rowkey in this case has changed to name.

A `GET` against the `users_optimized` for a known row key will be very fast:

```sql
hbase(main):019:0> get 'users_optimized', '0234567'
COLUMN        CELL
 f1:address   timestamp=1535718910795, value=73642 Cali Prairie, Dorthabury, PR 83524
 f1:dob       timestamp=1535718910795, value=2010-11-05
 f1:name      timestamp=1535718910795, value=Candace Bernier
 f1:phone     timestamp=1535718910795, value=(038)537-3097x47720
 f1:state     timestamp=1535718910795, value=PR
1 row(s)
Took 0.0308 seconds
```

If you don't have that key then hbase has no choice but to scan the whole table.

So what do you think happens when you scan the related table? By which criteria will the scan be the quickest?

Try it:

```sql
hbase(main):038:0> get 'users_name', 'Cassie Wilkinson'
```

Now what about a range scan? Try this:

```sql
scan 'users_name', {STARTROW => 'Cassie', ENDROW => 'Cassif'}
```

### Hive

Now let us look at joins in Hive.

>Note: You may have to do this first to make sure you have write permissions:

```console
[centos@ip-10-0-0-34 nosql]$ sudo su hive
[hive@ip-10-0-0-34 nosql]$ hdfs dfs -chmod 777 /warehouse/tablespace/managed/hive
[hive@ip-10-0-0-34 nosql]$ exit
[centos@ip-10-0-0-34 nosql]$
```

From the `data/nosql` directory copy several files to the `/tmp` directory:

```console
cp data.hive /tmp/.
cp orders.hive /tmp/.
```

Now run several beeline create tables and import:

```console
## Create the hive users table
beeline -n hive -u jdbc:hive2://localhost:10000 -e "CREATE  TABLE users(id int, name string, address string, dob string, phone string,state string) row format delimited stored as textfile;"
## Load the users data
beeline -n hive -u jdbc:hive2://localhost:10000 -e "load data local inpath '/tmp/data.hive' into table users"
## create the hive orders table. 
beeline -n hive -u jdbc:hive2://localhost:10000 -e "CREATE  TABLE orders(id int,total string) row format delimited stored as textfile;"
## Load the orders data 
beeline -n hive -u jdbc:hive2://localhost:10000 -e "load data local inpath '/tmp/orders.hive' into table orders"
```

Go into `hive`, using `beeline`:

```console
[centos@ip-10-0-0-34 nosql]$ beeline -n hive -u jdbc:hive2://localhost:10000
...
0: jdbc:hive2://master1.hdp.com:2181/default>
```

Now let's select a sample row:

```sql
0: jdbc:hive2://localhost:10000> select * from users limit 1;
INFO  : Compiling command(queryId=hive_20180831150529_0e30aa65-b4fb-4ca6-adc6-86d406578217): select * from users limit 1
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:users.id, type:int, comment:null), FieldSchema(name:users.name, type:string, comment:null), FieldSchema(name:users.address, type:string, comment:null), FieldSchema(name:users.dob, type:string, comment:null), FieldSchema(name:users.phone, type:string, comment:null), FieldSchema(name:users.state, type:string, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hive_20180831150529_0e30aa65-b4fb-4ca6-adc6-86d406578217); Time taken: 0.147 seconds
INFO  : Executing command(queryId=hive_20180831150529_0e30aa65-b4fb-4ca6-adc6-86d406578217): select * from users limit 1
INFO  : Completed executing command(queryId=hive_20180831150529_0e30aa65-b4fb-4ca6-adc6-86d406578217); Time taken: 0.001 seconds
INFO  : OK
+-----------+-------------------+------------------------------------------------+-------------+--------------+--------------+
| users.id  |    users.name     |                 users.address                  |  users.dob  | users.phone  | users.state  |
+-----------+-------------------+------------------------------------------------+-------------+--------------+--------------+
| 1         | Alexandre Sporer  | 24594 Emmitt Locks, Greenfelderview, MT 48128  | 2007-05-02  | 07683017318  | MT           |
+-----------+-------------------+------------------------------------------------+-------------+--------------+--------------+
1 row selected (0.258 seconds)
```

>Note: how fast was that? Did a `mapreduce` run?

Now when we run a join we suddenly have a lot more output:

```sql
select * from users join orders on users.id=orders.id limit 1;
```

And part of that output is INFO about the shuffle:

```console
INFO  :    SHUFFLE_BYTES: 2017486
INFO  :    SHUFFLE_BYTES_DECOMPRESSED: 3135824
INFO  :    SHUFFLE_BYTES_TO_MEM: 0
INFO  :    SHUFFLE_BYTES_TO_DISK: 0
INFO  :    SHUFFLE_BYTES_DISK_DIRECT: 2017486
INFO  :    SHUFFLE_PHASE_TIME: 1395
```

What about the speed of this query?

```console
1 row selected (5.056 seconds)
```

Why is the query that much slower?

### Prejoined Files

So lets do something to speed up the select. We are going to "pre-join" the data in two tables:

```sql
create table prejoined as
select users.id, users.name, users.address, users.dob,users.phone,users.state, collect_set(orders.total) as orderlist from users left join orders on users.id=orders.id group by users.id,users.name,users.address, users.dob,users.phone,users.state ;
```

>Note: copyable command:

```sql
create table prejoined as select users.id, users.name, users.address, users.dob,users.phone,users.state, collect_set(orders.total) as orderlist from users left join orders on users.id=orders.id group by users.id,users.name,users.address, users.dob,users.phone,users.state;
```

You should see the table and [vertices](https://books.google.com/books?id=XpDqDAAAQBAJ&pg=PA228&lpg=PA228&dq=hive+vertices&source=bl&ots=q7uJrdc7wQ&sig=wstMzONRjZQEGZr7C8CaEgzjxaU&hl=en&sa=X&ved=2ahUKEwjayua30ZfdAhUB26wKHXDrC8I4FBDoATADegQICBAB#v=onepage&q=hive%20vertices&f=false) created while running:

```console
----------------------------------------------------------------------------------------------
        VERTICES      MODE        STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  KILLED
----------------------------------------------------------------------------------------------
Map 3 .......... container     SUCCEEDED      1          1        0        0       0       0
Map 1 .......... container     SUCCEEDED      3          3        0        0       0       0
Reducer 2 ...... container     SUCCEEDED      2          2        0        0       0       0
----------------------------------------------------------------------------------------------
VERTICES: 03/03  [==========================>>] 100%  ELAPSED TIME: 15.14 s
----------------------------------------------------------------------------------------------
```

Now go and select a sample row from the prejoined table:

```sql
0: jdbc:hive2://localhost:10000> select * from prejoined where size(orderlist) > 0 limit 1;
...
INFO  : OK
+---------------+-------------------+------------------------------------------------+----------------+------------------+------------------+----------------------+
| prejoined.id  |  prejoined.name   |               prejoined.address                | prejoined.dob  | prejoined.phone  | prejoined.state  | prejoined.orderlist  |
+---------------+-------------------+------------------------------------------------+----------------+------------------+------------------+----------------------+
| 29            | Dr. Raegan Hoppe  | 293 Malcom Bypass, Marionmouth, MP 94721-1098  | 1990-05-19     | (667)615-4885    | MP               | ["648.6"]            |
+---------------+-------------------+------------------------------------------------+----------------+------------------+------------------+----------------------+
1 row selected (0.375 seconds)
```

And just to be sure let's see if any users are tied to more than 1 order:

```sql
0: jdbc:hive2://localhost:10000> select * from prejoined where size(orderlist) > 1 limit 1;
...
INFO  : OK
+---------------+-----------------+---------------------------------------------------+----------------+-------------------+------------------+----------------------+
| prejoined.id  | prejoined.name  |                 prejoined.address                 | prejoined.dob  |  prejoined.phone  | prejoined.state  | prejoined.orderlist  |
+---------------+-----------------+---------------------------------------------------+----------------+-------------------+------------------+----------------------+
| 5680          | Theodis Russel  | 1950 Mayer Dam Apt. 562, West Kenyatta, ID 83244  | 2005-07-24     | +23(1)3715682566  | ID               | ["676.38","788.82"]  |
+---------------+-----------------+---------------------------------------------------+----------------+-------------------+------------------+----------------------+
1 row selected (0.235 seconds)
```

And we see there are.

### HBase and the HBase Master UI

Let's go back to HBase. And lets create another table:

```sql
hbase(main):002:0> create 'user','cf1', SPLITS => ['g','m']
Created table user
Took 2.2917 seconds
=> Hbase::Table - user
```

Now let's put in some rows:

```sql
hbase(main):007:0> put 'user','a','cf1:name','test'
hbase(main):008:0> put 'user','k','cf1:name','test'
hbase(main):009:0> put 'user','r','cf1:name','test'
```

And the scan shows:

```sql
hbase(main):010:0> scan 'user'
ROW     COLUMN+CELL
 a      column=cf1:name, timestamp=1535736598392, value=test
 k      column=cf1:name, timestamp=1535736606360, value=test
 r      column=cf1:name, timestamp=1535736615460, value=test
3 row(s)
Took 0.0210 seconds
```

Now go to the HBase Master UI:

![image](https://user-images.githubusercontent.com/558905/44928794-f8d6f680-ad26-11e8-9d4d-039960e10e42.png)

See how your regions are split?

Now increment the reads:

```sql
hbase(main):024:0> get 'user','a'
COLUMN              CELL
 cf1:name           timestamp=1535736598392, value=test
1 row(s)
Took 0.0134 seconds
hbase(main):025:0> get 'user','r'
COLUMN              CELL
 cf1:name           timestamp=1535736615460, value=test
1 row(s)
Took 0.0042 seconds
```

And check the Metrics tab:

![image](https://user-images.githubusercontent.com/558905/44929318-ab5b8900-ad28-11e8-838a-a3d44f279f0b.png)

Now you see how HBase can be tuned by these region splits.


### Results



<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>