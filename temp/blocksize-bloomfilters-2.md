## Lab: Setting Blocksize and Enabling Bloomfilters

**Data directory**:  `~/data/users_orig.tsv`

**HBase tables**:    `user, userbackup, movie, moviebackup, customer`

Now you are going to set blocksize to improve access time and enable Bloom filters to use memory more efficiently. 

----

###	Set the Block size

Scan timeline table for rows 10000-14000:

		hbase> scan 'timeline', {STARTROW => '10000', ENDROW => '14000'}

A test scan took approximately 39.6 seconds.

Go to a web browser and localhost:60010. At the Region Servers section click the memory tab and note the used heap size.

Disable the table timeline:

		hbase> disable 'timeline'

Alter the table so the column family 'a' has a blocksize of 262144:

		hbase> alter 'timeline', {NAME => 'a', BLOCKSIZE => '262144'}

Enable the table timeline:

		hbase> enable 'timeline'

Run the scan twice again and note the time:

A test scan took approximately 39.2 seconds the first time and 38.2 the second time.

Again, go to the browser and note the used heap size

Disable the table timeline again and change block size to 524288, enable and run the scan two more times:

		hbase> disable 'timeline'
		hbase> alter 'timeline', {NAME => 'a', BLOCKSIZE => '524288'} hbase> enable 'timeline'

A test run took 38.17 the first time and 38.1 seconds the second time.

Go to the browser and note the used heap size once again

Enable Bloom filters and note access time

Use get to retrieve row 1201 from the timeline table

		hbase> get 'timeline', '1201'

A test run took 0.24 seconds with a memory heap used of 153m

Disable the table and alter it to use a Bloom filter at the row level. Then enable the table again.

		hbase> disable 'timeline'
		hbase> alter 'timeline', {NAME => 'a', BLOOMFILTER => 'ROW'} 
		hbase> enable 'timeline'

Run the get statement again.

		hbase> get 'timeline', '1201'

A test run took .009 seconds but used heap memory went to 156m

Repeat the process again setting Bloom filter at the row and column level (ROWCOL).

A test get took .011 (an increase) but heap usage went down to 136m

Experimenting with different gets on different keys, you should be coming in at aroung .009 to .007 filtering on the row.

When filtering on both row and column, the retrieval time goes up to nearly .01 each time (if you experiment) so there are no advantageous in our use case.


### Summary

You should have been able to observe slight improvements in access time by adjusting blocksize and using bloomfilters. These observations are based on a tiny amount of data. Consider the implications when they are used with terabytes of data.
