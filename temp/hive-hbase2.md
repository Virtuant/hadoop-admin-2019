## Lab: Hive and HBase

**Objective**: Associate HBase tables with Hive and then use Hive to run queries on them. T
his will allow you to manage the pre-existing HBase table with Hive. 
You will also import data via Sqoop, which is another ecosystem project. 
It allows relational database tables to be imported into HBase and other Hadoop technologies.

**Exercise directory**: `~/data`

**Data directory**: `~/data/NYSE_daily_prices_A.csv`

**HBase tables**:  `movierating, movie, user`

**Hive tables**:   `movierating, movie, user`


----

### Importing a Table to Hive via Sqoop

If you haven't already, import the movie rating table through Sqoop: `~/data/hbase/importMovieRatings.sh`

Start Hive shell.

    ```console
    hive
    ```
    
Add Hive’s HBase handler, and the HBase jar:

    ```console
    hive> add jar /usr/hdp/current/hive-client/lib/hive-hbase-handler.jar;
    hive> add jar /usr/hdp/current/hbase-client/lib/hbase-hadoop-compat.jar;
    ```

Create an external Hive table to manage the movierating table you imported with Sqoop:

    - :key refers to the HBase row key
    - rating:movieid refers to column family rating and column qualifier movieid
    - rating:rating refers to the column family rating and the column qualifier rating

    ```console
    hive> CREATE EXTERNAL TABLE movierating (rowkey string, userid int, movieid int, rating int) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES ("hbase.columns.mapping" =  ":key,rating:userid,rating:movieid,rating:rating") TBLPROPERTIES ("hbase.table.name" = "movierating");
    ```

Scan the table to make sure its successful:
    
    ```console
    hive> select * from movierating limit 10;
    ```
    
Exit hive.
    
### Create an external Hive table for the movie:

    ```console
	sqoop import --connect jdbc:mysql://localhost/movielens --hbase-table movie --column-family movie --username root --password hadoop --query 'select CAST(FLOOR(RAND() * 100000000000000000000) as char) as rowkey, movie.id, movie.name, movie.year from movielens.movie WHERE $CONDITIONS' --split-by movie.id --hbase-row-key rowkey --hbase-create-table --driver com.mysql.jdbc.Driver
	```
	
Create an external Hive table for the user:

    ```console
	sqoop import --connect jdbc:mysql://localhost/movielens --hbase-table user --column-family movie --username root --password hadoop --query 'select CAST(FLOOR(RAND() * 100000000000000000000) as char) as rowkey, user.id, user.gender, user.age, user.occupationid, user.zip from movielens.user WHERE $CONDITIONS' --split-by user.id --hbase-row-key rowkey --hbase-create-table --driver com.mysql.jdbc.Driver
	```

In the HBase shell, issue the following command to see the column qualifiers for the info column family of table user:

    ```console
    scan 'user', {LIMIT => 1}
    ```
    
and the table movie:
    
    ```console
    scan 'movie', {LIMIT => 1} 
    ```

### Select Rows

Select user id 923 from the user table.

Select movie id 1213 from the movie table.

Select the following movie ids from the movie table in one query:

    ```console
    45
    67        
    78
    190        
    2001
    ```

### Advanced Selects

Hive has built-in functions, just like many other RDBMSs. There are functions such as  
`COUNT(), AVG(), MIN(), MAX(), and SUBSTR()`. 

The `SUBSTR()` function returns the substring, or slice, of the byte array beginning at the start position and continuing to the end of the string. For example, `SUBSTR('foobar', 4)` returns bar.

To use a function, simply call the function on a column. Here is in an example showing how to use the count function to total the number of users in the users table:

```sql
hive> SELECT COUNT(user.rowkey) FROM user;
```

Use a query with a function to find the average of every user rating in the movierating

Output the oldest and newest year of a movie where the year is not 0 (unknown).

Output the rows that match the following criteria:

    - The movie’s genre is 'Comedy'
    - The movie was made in the 1990s
    - The movie's average rating is in the 4s

Output the distribution of the movies by the decade they came out.

### Using Joins

Output the movie name, movie year, and user rating ordered by the movie name for every movie that matches the following criteria:

- The user’s id is 517
- The rating given by the user is over 4

### Secondary Index

Hive can both read from and **write to** HBase. Hive can insert new rows into a previously created table. The table will need to be created in HBase. An external table will also be created in Hive. The syntax looks like this:

```sql
INSERT INTO TABLE othertable;
SELECT * FROM table;
```

Create a secondary index on the HBase movie table called `moviename` with a single column family called `info`. Have the row key be `moviename` or the actual name of the movie. The value should be a single column called `movierow` and its value should be the original row key in the movie table.

Verify that the insert is correct by running a limited scan on `moviename` table in the HBase shell.

Create a composite secondary index on the movie table called `moviecomp` with a single column family called `info`. Have the row key be `<movieyear><moviegenre><moviename>`. The value should be a single column called `movierow` and its value should be the original row key in the movie table. Be sure to separate the values in the composite with a colon. Use the `concat()` method to concatenate everything together.

Verify that the insert is correct by running a limited scan on moviename table in the HBase shell.

### Summary


