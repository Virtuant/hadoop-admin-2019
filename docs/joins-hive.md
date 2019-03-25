## Joining Data Sets with Hive

**Data Files:** ~/data/stocks

In this exercise you will practice setting various Hive configuration options.

----

### Joining Datasets and Contents Analysis PriceData load into Hive

1.	Go to your data stocks directory

2.	Check contents of the files using the `head` command on local file system. Note these two local files:

```console
$ head NYSE_daily_prices_F.csv
$ head NYSE_dividends_C.csv
```

3.	Verify the two files you have put into HDFS (if not there go ahead and put them there):

```console
$ hdfs dfs -ls [path]/*.csv
```

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) remember the `[path]` is optional and selected by you

4.	Check that the `prices_f` table is still in Hive, load it if not:

```console
$ hdfs dfs -ls /apps/hive/warehouse
```

### Dividend Data load into Hive

1.	Let's create a new hive file called `aggregate.hive` at the Linux prompt:

```console
$ touch aggregate.hive
```

2.	Use a text editor to add below lines to create the `dividend_c` table:

```console
CREATE TABLE dividend_c (market string, symbol string, 
trade_date string, dividend float) 
row format delimited fields terminated by ',';
```

3.	To load the data into the table, add below command to the file:

```console
LOAD DATA INPATH '[path]/[filename].csv' 
OVERWRITE INTO TABLE dividend_c;
```

4.	Now we need to create a table to have the data sets joined. Let's add below command to the file to build the table named `stock_aggregate`:

```console
CREATE TABLE stock_aggregate (symbol string, year string, high float, low float,
average_close float, total_dividends float);
```

Save the file.

5.	View the file:

```console
$ cat aggregate.hive
```

Make sure it has the correct commands as above.

6.	Now execute the hive file:

```console
$ hive -f aggregate.hive
```

7.	Verify that data exists in the `dividend_c` table in hive:

```console
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

8.	Verify the stock_aggregate table:

```console
hive>  describe stock_aggregate;
OK
symbol	string
year	string
high	float
low	float
average_close	float 
total_dividends	float
```

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png)  Question: do you see any data in the stock_aggregate table? Answer: No

### Now join the price and dividend data in Hive

1.	Create a new Hive file at the Linux prompt:

```console
$ touch price_dividend_join.hive
```

2.	Edit the price_dividend_join.hive file with a text editor, first adding the data insert statement:

```console
insert overwrite table stock_aggregate
```

3.	In the same line add below command to the above statement:
 
```console
select p.symbol, year(p.dates), max(.price_high), min(p.price_low), avg(p.price_close), sum(d.dividend) from prices_fp
```

4.	Now we need to do a join to the dividend_c table

```console
left outer join dividend_c d
```

5.	Add the join:

```console
on(p.symbol=d.symbol and p.dates=d.trade_date)
```

6.	the below will need to be added to get the aggregate result:

```console
group by p.symbol,year(p.dates);
```

7.	Verify the file:

```console
$ more price_dividend_join.hive
insert overwrite table stock_aggregate select p.symbol,year(p.dates), max(p.price_high), min(p.price_low), avg(p.price_close), sum(d.dividend) from prices_fp left outer join dividend_c d on (p.symbol=d.symbol and p.dates=d.trade_date) group by p.symbol, year(p.dates);
```

### Execute the Hive script

1.	Now execute the script:

```console
$ hive -f price_dividend_join.hive
...
Total MapReduce CPU Time Spent: 7 seconds 910 msec 
OK
```

2.	How long does this job run?

Time taken: 43.654 seconds

Note the time taken might be different in your environment.
 

3.	How many Map and Reduce jobs executed? 1 Map and 1 Reduce job

### View the Aggregate Table Contents

1.	Verify a file is generated:

```console
$ hdfs dfs -ls
000000_0
```

2.	Now use the text command to check the data again and get a row counts:

```console
$ hdfs dfs -text /apps/hive/warehouse/stock_aggregate/ 000000_0 | wc -l
1748
```

### View the aggregate table contents in Hive

1.	Retrieve data from the table:

```console
$ hive hive>
select * from stock_aggregate;
....
FXEN 200 68.53 3.84 5.27 79284 NULL
FXEN 200 710.6 4.6 47.69 3825 NULL
FXEN	200	88.6	81.9	55.24	664	NULL
FXEN	200	94.9	72.1	23.27	0635	NULL
FXEN 201 03.2 32.8 43.03 16 NULL symbol NULL NULL NULL NULL NULL
```

2.	How many rows were returned from the above query? Fetched: 1748 row(s)
Does it match the count from Step 4.4?
Answer: Yes

3.	Use count command to get the total rows:
 
```console
hive>
select count(*) from stock_aggregate;
```

What is the result? Answer: 1748

END
