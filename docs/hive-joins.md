## Joining Data Sets with Hive

**Data Files:** ~/data/stocks

In this exercise you will practice setting various Hive configuration options.

----

### Joining Datasets and Contents Analysis PriceData load into Hive

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1.	Go to your data stocks directory</h4>

Check contents of the files using the `head` command on local file system. Note these two local files:

```console
$ head NYSE_daily_prices_F.csv
$ head NYSE_dividends_C.csv
```

Verify the two files you have put into HDFS (if not there go ahead and put them there):

```console
$ hdfs dfs -ls [path]/*.csv
```

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) remember the `[path]` is optional and selected by you

4.	Check that the `prices_f` table is still in Hive, load it if not:

```console
$ hdfs dfs -ls /apps/hive/warehouse
```

### Dividend Data load into Hive

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Let's create a new hive file</h4>
 
Call it `aggregate.hive` at the Linux prompt:

```console
$ touch aggregate.hive
```

Use a text editor to add below lines to create the `dividend_c` table:

```sql
CREATE TABLE dividend_c (market string, symbol string, 
trade_date string, dividend float) 
row format delimited fields terminated by ','
stored as textfile;
```

To load the data into the table, add below command to the file:

```console
LOAD DATA INPATH '[path]/[filename].csv' 
OVERWRITE INTO TABLE dividend_c;
```

Now we need to create a table to have the data sets joined. Let's add below command to the file to build the table named `stock_aggregate`:

```console
CREATE TABLE stock_aggregate (symbol string, year string, high float, low float,
average_close float, total_dividends float);
```

Save the file.

View the file:

```console
$ cat aggregate.hive
```

Make sure it has the correct commands as above.

Now execute the hive file:

```console
$ hive -f aggregate.hive
```

Verify that data exists in the `dividend_c` table in hive:

```sql
hive>  select * from dividend_c limit 10;
Results: exchange	
stock_symbol	date	
NULL
NYSE	CY	2008-09-30	16.418	
NYSE	CY	2008-09-29	0.0	
NYSE	CPO	2009-12-30	0.14	
NYSE	CPO	2009-09-28	0.14	
NYSE	CPO	2009-06-26	0.14	
NYSE	CPO	2009-03-27	0.14	
NYSE	CPO	2009-01-06	0.14	
NYSE	CPO	2008-09-30	0.14	
NYSE	CPO	2008-06-24	0.14	
Time taken:	0.266 seconds, Fetched: 10	row(s)
```

Verify the stock_aggregate table:


```sql
hive>  describe stock_aggregate;
OK
symbol	string
year	string
high	float
low	float
average_close	float 
total_dividends	float
```

> ![note](https://user-images.githubusercontent.com/558905/40528492-37597500-5fbf-11e8-96a1-f4d206df64ab.png)  Question: do you see any data in the stock_aggregate table? Answer: No

### Now join the Price and Dividend data in Hive

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3.	Create a new Hive file at the Linux prompt:</h4>

```console
$ touch price_dividend_join.hive
```

Edit the price_dividend_join.hive file with a text editor, first adding the data insert statement:

```sql
insert overwrite table stock_aggregate
```

In the same line add below command to the above statement:
 

```sql
select p.symbol, year(p.dates), max(.price_high), min(p.price_low), avg(p.price_close), sum(d.dividend) from prices_fp
```

Now we need to do a join to the dividend_c table

```sql
left outer join dividend_c d
```

Add the join:


```sql
on(p.symbol=d.symbol and p.dates=d.trade_date)
```

The below will need to be added to get the aggregate result:

```sql
group by p.symbol,year(p.dates);
```

Verify the file:

```console
$ more price_dividend_join.hive
insert overwrite table stock_aggregate select p.symbol,year(p.dates), max(p.price_high), min(p.price_low), avg(p.price_close), sum(d.dividend) from prices_fp left outer join dividend_c d on (p.symbol=d.symbol and p.dates=d.trade_date) group by p.symbol, year(p.dates);
```

### Execute the Hive script

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4.	Now execute the script:</h4>

```console
$ hive -f price_dividend_join.hive
...
Total MapReduce CPU Time Spent: 7 seconds 910 msec 
OK
```

How long does this job run?

Time taken: 43.654 seconds


> ![note](https://user-images.githubusercontent.com/558905/40528492-37597500-5fbf-11e8-96a1-f4d206df64ab.png) the time taken might be different in your environment.
 

How many Map and Reduce jobs executed? 1 Map and 1 Reduce job

### View the Aggregate Table Contents

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>5.	Verify a file is generated:</h4>

```console
$ hdfs dfs -ls
000000_0
```

Now use the text command to check the data again and get a row counts:

```console
$ hdfs dfs -text /apps/hive/warehouse/stock_aggregate/000000_0 | wc -l
1748
```

### View the aggregate table contents in Hive

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>6.	Retrieve data from the table:</h4>

```sql
$ hive> select * from stock_aggregate;
....
FXEN 200 68.53 3.84 5.27 79284 NULL
FXEN 200 710.6 4.6 47.69 3825 NULL
FXEN	200	88.6	81.9	55.24	664	NULL
FXEN	200	94.9	72.1	23.27	0635	NULL
FXEN 201 03.2 32.8 43.03 16 NULL symbol NULL NULL NULL NULL NULL
```

How many rows were returned from the above query? Fetched: 1748 row(s)
Does it match the count from Step 4.4?
Answer: Yes

Use count command to get the total rows:

```sql
hive> select count(*) from stock_aggregate;
```

What is the result? Answer: 1748

### Results

Great! We joined several tables in hive.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>
