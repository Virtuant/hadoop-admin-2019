## Lab: More Exploring HBase

**HBase tables**:     `sometable`

## About This Lab

### Objective:
Use the HBase Shell to explore the `hbase:meta` table.

### Successful outcome:
Have a basic understanfing of HBase shell and meta table.

### Lab Steps

In this exercise you will use more basic commands in HBase Shell to explore HBase. 

----

1. Enter HBase Shell

		[root@sandbox ~]# hbase shell
		hbase(main):001:0>

1. Create a table in HBase

		hbase(main):007:0> create 'sometable','ch3'

1. Now check the tables:

		hbase(main):008:0* list

1. Verify the new table:

		hbase(main):012:0> describe 'sometable'
		DESCRIPTION          ENABLED
		'sometable', {NAME => 'ch3', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATI true ON_SCOPE => '0', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL => '2147483647', KEEP_DELETED_CELLS => 'false', BLOCKSIZE => '65536', IN_MEMORY => 'false', BLOCKCACHE => 'true'}
		1 row(s) in 0.0750 seconds

Review the output and notice the column ('ch3'), parameter properties such as COMPRESSION, DATA_BLOCK_ENCODING, whether is kept in memory...

1. Insert data using PUT command:

		hbase(main):004:0> put 'sometable','row1','ch3','value1'
		0 row(s) in 0.4250 seconds
		hbase(main):005:0> put 'sometable','row2','ch3','value1'
		0 row(s) in 0.0120 seconds
		hbase(main):006:0> put 'sometable','row1','ch3','value2'
		0 row(s) in 0.0110 seconds
		hbase(main):007:0> put 'sometable','row1','ch3','value3'
		0 row(s) in 0.0070 seconds

1. View the table data using SCAN:

		hbase(main):008:0> scan 'sometable'
		ROW             COLUMN+CELL
		 row1           column=ch3:, timestamp=1408499993409, value=value3
		 row2           column=ch3:, timestamp=1408499977588, value=value1
		2 row(s) in 0.1020 seconds

1. This time view the data using SCAN with versions:

		hbase(main):011:0> scan 'sometable', { VERSIONS => 3}
		ROW             COLUMN+CELL
		 row1           column=ch3:, timestamp=1408499993409, value=value3
		 row2           column=ch3:, timestamp=1408499977588, value=value1
		2 row(s) in 0.0200 seconds

1. Use Count command to determine the number of rows in a table:

		hbase(main):002:0> count 'sometable'
		2 row(s) in 0.4020 seconds
		=> 2

1. Retrieve data using GET:

		hbase(main):012:0> get 'sometable','row1'
		COLUMN             CELL
		 ch3:              timestamp=1408499993409, value=value3
		1 row(s) in 0.0530 seconds

1. Use GET command to check on row2:

		hbase(main):003:0> get 'sometable','row2'
		COLUMN             CELL
		 ch3:              timestamp=1408499977588, value=value1
		1 row(s) in 0.0110 seconds

1. Now use PUT command on row2:

		hbase(main):004:0> put 'sometable','row2','ch3','value2'
		0 row(s) in 0.3340 seconds

1. Retrieve the value again:

		hbase(main):005:0> get 'sometable','row2'

1. Test the row counts:

		hbase(main):006:0> count 'sometable'
		2 row(s) in 0.2290 seconds
		=> 2

1. Run the delete command:

		hbase(main):020:0> delete 'sometable','row1','ch3'
		0 row(s) in 0.1990 seconds

1. We can verify the data using SCAN command:

		hbase(main):021:0> scan 'sometable'
		ROW                COLUMN+CELL
		 row2              column=ch3:, timestamp=1409577011732, value=value2
		1 row(s) in 0.0750 seconds

1. Before we disable the table, let's run a count on the table:

		hbase(main):022:0> count 'sometable'
		1 row(s) in 0.0320 seconds
		=> 1

1. Execute the disable now:

		hbase(main):007:0> disable 'sometable'
		0 row(s) in 1.7060 seconds

1. Do the count command again:

		hbase(main):008:0> count 'sometable'

1. Now enable the table again:

		hbase(main):025:0> enable 'sometable'
		0 row(s) in 0.8730 seconds

1. Use count to check the table again:

		hbase(main):026:0> count 'sometable'

1. Check the table again:

		hbase(main):010:0> list
		TABLE
		...
		sometable
		...

1. Now use the DROP command to get rid of the table:

		hbase(main):011:0> drop 'sometable'
		0 row(s) in 0.8290 seconds

1. To verify use the list command again:

		hbase(main):012:0> list
