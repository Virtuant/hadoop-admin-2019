## Lab: NoSql and Performance Comparisons

**Objective**: Let's do a little more with HBase to see how the Regions make it perform better.

**HDFS paths:** `/tmp` 

**Data Source**: `users.csv`

In this exercise you will use the HBase Shell to see how performance works.

----

So do this simple table create:

```sql
hbase(main):001:0> create 't1','f1', SPLITS=>['m']
Created table t1
Took 2.8872 seconds
=> Hbase::Table - t1
```

Put some data in:

```sql
base(main):006:0> put 't1','riley','f1:name', 'barry'
Took 0.1688 seconds
```

Scan the table:

```sql
hbase(main):007:0> scan 't1'
ROW                              COLUMN+CELL
 riley                           column=f1:name, timestamp=1535500766655, value=barry
1 row(s)
Took 0.0313 seconds
```

Put in this record:

```sql
hbase(main):018:0> put 't1','jane','f1:name', 'doe'
Took 0.0117 seconds
```

Scan again:

```sql
hbase(main):019:0> scan 't1'
ROW                                COLUMN+CELL
 jane                              column=f1:name, timestamp=1535500964402, value=doe
 riley                             column=f1:name, timestamp=1535500766655, value=barry
2 row(s)
Took 0.0080 seconds
```

Now go to back into HBase's Master UI:

![image](https://user-images.githubusercontent.com/558905/44757979-e2028b00-aaff-11e8-9363-345ba36d6169.png)


Now if you scroll down you will see that the table is split:

![image](https://user-images.githubusercontent.com/558905/44758004-fcd4ff80-aaff-11e8-8740-72e0b01545d6.png)

and `m` is the split point. The starting and ending key is set to `m`.


Now go to the Request Metrics page:

![image](https://user-images.githubusercontent.com/558905/44786841-66d2c080-ab63-11e8-801e-6973b348eafb.png)

And you see the number of requests in each table and region.

### Import Data

Now we're going to put some data into MySQL (as an example):

```console
 [centos@ip-10-0-0-34 data]$ mysql -p -u root  < users.sql
 Enter password: 
 [centos@ip-10-0-0-34 data]$ 
```

>Note: just enter when prompted for `password`

You can query the MySQL data if you want to.

Now go into HBase and create the target table with key:


```sql
hbase(main):002:0> create 'users','id'
Created table users
Took 1.3353 seconds                                                                                            
=> Hbase::Table - users
```

And now we'll use the import Hadoop feature called `sqoop`. [Here's](https://sqoop.apache.org/docs/1.99.3/Sqoop5MinutesDemo.html) a little five-minute demo.

Now do the import:

```sql
 [centos@ip-10-0-0-34 data]$ sqoop import 
    --connect jdbc:mysql://localhost/user_data 
    --username root 
    --table users 
    --hbase-table users 
    --column-family id 
    --hbase-row-key id 
    --where "state='CA'" 
    -m 3
```

>Note: the `-m` param is the number of mappers to use


Now scan `users` table in HBase:

```sql
 scan 'users'
```

### Lookup HBase by using Filters

Get 1 row:

```sql
hbase(main):009:0> get 'users', 99992
COLUMN            CELL                                                                              
 id:address       timestamp=1536235155902, value=2416 Durgan Springs Suite 807, West Windychester,
                  CA 68248-6281                                                                      
 id:dob           timestamp=1536235155902, value=1994-03-04                                         
 id:name          timestamp=1536235155902, value=Ephram Gulgowski                                   
 id:phone         timestamp=1536235155902, value=746-572-2726x952\x0D                               
 id:state         timestamp=1536235155902, value=CA                                                 
1 row(s)
Took 0.0163 seconds  
```

Now, a table Scan:

```sql
 scan 'users', { COLUMNS => 'id:name', FILTER => "ValueFilter( =, 'regexstring:Alden' )" }
 scan 'users', { COLUMNS => 'id:name', FILTER => "ValueFilter( =, 'regexstring:Johnson' )" }
```

## Hive commands

Let's look at Hive as a comparison:

```console
 [centos@ip-10-0-0-34 data]$ beeline -n hive -u jdbc:hive2://localhost:10000
```

Make some adjustments:

```sql
set mapreduce.job.reduces=2;
set hive.auto.convert.join = false;
```

Now this will do a full table scan in hive:

```sql
select * from users where id=99992;
```

> Note: how fast was the scan?

What about a full Table Scan creating a small Shuffle and aggregation:

```sql
select count(*) from users;
```

### Table Scan=>larger Shuffle

```sql
select avg(id) from users where state < 'CA' group by state;
```

And a Join that leads to scan + shuffle + hdfs write:

```sql
insert overwrite directory '/tmp/hive_output' select * from users join orders on users.id=orders.id ;
```

Now show some output:

```console
hdfs dfs -cat /tmp/hive_output/0* | grep 669410 | less
```
### Results

So you see how HBase compares with Hive on simple queries. Good Job.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>
