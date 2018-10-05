# Links to Possible most current labs to Replace this one:

https://maprdocs.mapr.com/52/Hive/HiveHBaseIntegration-GettingStarted.html

https://acadgild.com/blog/integrating-hive-hbase/

https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration

https://www.quora.com/How-can-I-transfer-data-from-Hive-external-table-to-Hbase

## Lab: HBase and Hive Integration

Understand how HBase and Hive integration. You will complete data storage in HBase from Hive table data.
## Objective:
Understand how HBase and Hive integration.

### File locations:
/etc/hbase/conf/hbase_env.sh, /etc/hbase/conf/hbase- site.xml
/etc/zookeeper/conf/zookeeper_env.sh, /etc/zookeeper/conf/zoo.cfg
### Successful outcome:
You will:
Complete data storage in HBase from Hive table data.
### Lab Steps

Complete the setup guide

----

1. Start hive shell

		# hive

1. Create a Hive table

		hive> CREATE TABLE hbase_test_1(key int, value string) 
		STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
		WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,cf1:val")
		TBLPROPERTIES ("hbase.table.name" = "test_1");
		OK
		Time taken: 11.565 seconds

1. Validate the table in Hive

		hive> describe hbase_test_1;
		OK
		key                     int                     from deserializer
		value                   string                  from deserializer
		Time taken: 1.795 seconds, Fetched: 2 row(s)

1. Create a lookup table in Hive:

		hive> CREATE TABLE stocks (exchg STRING, symbol STRING, priceDate STRING, 
		open FLOAT, high FLOAT, low FLOAT, close FLOAT, volume INT, adjClose FLOAT) 
		ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
		
	And load data into the table:
	
		hive> LOAD DATA LOCAL INPATH 'data/NYSE_daily_prices_A.csv' OVERWRITE INTO TABLE stocks;
	
	Now print to make sure its loaded:
	
		hive> select * from stocks;

1. Exit Hive

		hive> exit;

### Validate the table in HBase

1. Enter HBase command line interface:

		# hbase shell

1. Use LIST command to check:

		hbase(main):005:0> list
		TABLE
		...
		test_1
		...

1. Validate the table in HBase

		hbase(main):007:0> describe "test_1"
		DESCRIPTION                   ENABLED
		 'test_1', {NAME => 'cf1', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATI true ON_SCOPE => '0', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL => '2147483647', KEEP_DELETED_CELLS => 'false', BLOCKSIZE => '65536', IN_MEMORY => 'false', BLOCKCACHE => 'true'}
		1 row(s) in 0.0970 seconds

1. Now use SCAN to find if data exists

		hbase(main):008:0> scan 'test_1'
		ROW               COLUMN+CELL
		0 row(s) in 0.0060 seconds

	As it shows, there is no data in the table

1. Exit HBase shell

		hbase(main):024:0> quit

### Populate the table in Hive with Tez

1. Start Hive shell

		# hive

1. Run below command to set Hive engine as Tez 

		hive> set hive.execution.engine=tez;

1. Execute below HiveQL to populate the table

		hive> INSERT OVERWRITE TABLE hbase_test_1 SELECT volume, symbol FROM stocks WHERE volume>10000000;
		Query ID = root_20140830123030_3fee9010-e712-4c44-89ec-1261c220e424
		Total jobs = 1
		Launching Job 1 out of 1
		Status: Running (application id: application_1409394057604_0003)
		Map 1: -/-
		Map 1: 0/1
		Map 1: 0/1
		Map 1: 0/1
		Map 1: 0/1
		Map 1: 0/1
		Map 1: 0/1
		Map 1: 1/1
		Status: Finished successfully
		OK
		Time taken: 24.005 seconds

1. Now check the data in Hive:

		hive> select * from hbase_test_1;
		OK
		16298100        FNFG
		16982800        FNFG
		23728300        FNFG
		26681300        FNFG
		Time taken: 0.342 seconds, Fetched: 4 row(s)

1. Now exit Hive session:

		hive> exit;

### Verify the table data in HBase

1. Enter HBase session:

		# hbase shell

1. Use SCAN to view the data:

		hbase(main):009:0> scan 'test_1'
		ROW              COLUMN+CELL
		16298100         column=cf1:val, timestamp=1409427074706, value=FNFG
		16982800         column=cf1:val, timestamp=1409427074706, value=FNFG
		23728300         column=cf1:val, timestamp=1409427074706, value=FNFG
		26681300         column=cf1:val, timestamp=1409427074706, value=FNFG
		4 row(s) in 0.1110 seconds

1. Use DESCRIBE to find it table raw_data exists:

		hbase(main):002:0> describe 'raw_data'
		ERROR: Unknown table raw_data

1. Run below command to create the table:

		hbase(main):003:0> create 'raw_data', 'trades'
		...
		0 row(s) in 4.6990 seconds
		=> Hbase::Table - raw_data

1. Verify the table:

		hbase(main):002:0> describe 'raw_data'
		DESCRIPTION                     ENABLED
		'raw_data', {NAME => 'trades', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW true', REPLICATION_SCOPE => '0', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS=> '0', TTL => '2147483647', KEEP_DELETED_CELLS => 'false', BLOCKSIZE => '65536', IN_MEMORY => 'false', BLOCKCACHE => 'true'}
		1 row(s) in 0.1450 seconds

### Populate data in HBase

1. Use PUT command to insert the data (hbase prompt missing to enable copying):

		put 'raw_data', 'row1', 'trades:traderid', 'trader1'
		put 'raw_data', 'row1', 'trades:symbol', 'stock1'
		put 'raw_data', 'row1', 'trades:rating', 'High'
		put 'raw_data', 'row2', 'trades:traderid', 'trader2'
		put 'raw_data', 'row2', 'trades:symbol', 'stock1'
		put 'raw_data', 'row2', 'trades:rating', 'Low'
		put 'raw_data', 'row3', 'trades:traderid', 'trader2'
		put 'raw_data', 'row3', 'trades:symbol', 'stock2'
		put 'raw_data', 'row3', 'trades:rating', 'Low'
		put 'raw_data', 'row4', 'trades:traderid', 'trader2'
		put 'raw_data', 'row4', 'trades:symbol', 'stock4'
		put 'raw_data', 'row4', 'trades:rating', 'High'
		put 'raw_data', 'row5', 'trades:traderid', 'trader3'
		put 'raw_data', 'row5', 'trades:symbol', 'stock3'
		put 'raw_data', 'row5', 'trades:rating', 'Medium'
		put 'raw_data', 'row6', 'trades:traderid', 'trader3'
		put 'raw_data', 'row6', 'trades:symbol', 'stock4'
		put 'raw_data', 'row6', 'trades:rating', 'High'

1. Now use SCAN to view the data:

		hbase(main):023:0> scan 'raw_data'
		ROW               COLUMN+CELL
		 row1             column=trades:rating, timestamp=1409515999581, value=High
		 row1             column=trades:symbol, timestamp=1409515976501, value=stock1
		 row1             column=trades:traderid, timestamp=1409515950335, value=trader1
		 row2             column=trades:rating, timestamp=1409516016464, value=Low
		 row2             column=trades:symbol, timestamp=1409516016441, value=stock1
		...
		6 row(s) in 0.2080 seconds

1. Exit HBase session:

		hbase(main):024:0> quit

### Create a Hive table to integrate with HBase

1. Start a Hive session:

		# hive

1. Use DESCRIBE to make sure the hive table name hbase_raw_data does not exist:

		hive> describe hbase_raw_data;
		FAILED: SemanticException [Error 10001]: Table not found hbase_raw_data

1. Execute the below command to create the table:

		hive> CREATE EXTERNAL TABLE hbase_raw_data
		(key string, traderid string,symbol string,rating string)
		STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
		WITH SERDEPROPERTIES
		("hbase.columns.mapping" = ":key,trades:traderid,trades:symbol,trades:rating")
		TBLPROPERTIES ("hbase.table.name" = "raw_data");
		OK
		Time taken: 5.465 seconds

3.4 Verify the table structure:

		hive> describe hbase_raw_data;
		OK
		key                     string                  from deserializer
		traderid                string                  from deserializer
		symbol                  string                  from deserializer
		rating                  string                  from deserializer
		Time taken: 0.874 seconds, Fetched: 4 row(s)

1. Now view the data in the hive table:

		hive> select * from hbase_raw_data;
		OK
		row1    trader1 stock1  High
		row2    trader2 stock1  Low
		row3    trader2 stock2  Low
		row4    trader2 stock4  High
		row5    trader3 stock3  Medium
		row6    trader3 stock4  High
		Time taken: 1.186 seconds, Fetched: 6 row(s)

1. Exit Hive:

		hive> exit;
