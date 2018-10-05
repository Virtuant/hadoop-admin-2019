Commonlounge
Big Data
About
Go Pro
How to Contribute
Sign up · Log in
TutorialBig DataLast updated 5 months ago
Part of course: The Hands-On Guide to Hadoop and Big Data
HBase Tutorial — Hadoop and NoSQL Part 1[ Edit ]
What is NoSQL?
A NoSQL database is a type of database that stores and retrieves data differently than a traditional relational database. NoSQL databases are also known as non relational databases, or as Not-only-SQL databases because they can have a SQL-like language that is used to query data.
NoSQL databases grew in popularity with the emergence of companies such as Google and Amazon because of the requirements of real time web applications combined with massive amounts of data (Big Data). It's simplistic design allows horizontally scalability across multiple nodes easily, resulting in high availability. They also offer a lot of options when it comes to how the data is stored and retrieved — data can be stored in key value pairs, wide column, document, and graphs, providing a great deal of flexibility.
Why does Hadoop need NoSQL?
Hadoop has some limitations when it comes to processing and retrieval of data. HDFS is really good at sequential querying of data. One thing HDFS cannot do is query data in a random fashion. For example, if Hadoop was to look up a word in the dictionary, it would start from the beginning and go through every page to find the right word. If you had a NoSQL database, you could “randomly” go to a page and start looking or go to a certain letter and start looking there. This is why HBase was created for the Hadoop ecosystem.
What is a columnar, or column-based database?
A columnar database stores tables by column instead of by row. Imagine, taking a relational database table and turning it on its side.
Imagine a user table such as the one below:

A user table
A row oriented database might store this table row by row keyed by the rowId
1:512; Williams; Jason, a@gmail.com
2:344; Miller; Kelly, b@gmail.com
...
However, a column oriented database will clump the columns together:
512:1, 344:2, 231:3, 983:4
Williams:1, Miller:2, Wilson:3, Brown:4
Jason:1, Kelly:2, John:3, Nancy:4
...
Note that in the row-oriented example, rowId is the primary key, whereas in the column-oriented example, the data itself is the primary key.
Choosing which type to use depends on whether it is more likely that an entire row needs to be fetched when doing queries (which row-based databases are better at) or just a subset of the data or a few columns.
What is HBase?
HBase is a columnar database that is built on top of HDFS (thereby inheriting the distributed nature of HDFS). It is very similar to Google’s Big Table and is designed to give the end user random access across huge sets of data quickly.
The schema of the table defines only column families, which are a collection of columns. A column is a collection of key value pairs, does not need to defined in the schema, and can be created on the fly.
HBase Example Schema
Let’s look at an example of what some data inside of an HBase table looks like:

Row Key, Customer Information, and Customer Property are examples of Column Families. customerId, name, age, car type, and car color are examples of columns.
HBase Features
Since it is based on HDFS, HBase has the ability to scale horizontally and to interact with Hadoop’s processing power as a source and destination.
Also, since HBase is a Hadoop application:
It inherits the native fault tolerance that comes with Hadoop.
It has a native java api that can be used to interact with it.
It is provided with data replication so that data is never lost.
HBase does provide consistent reads and writes and should be looked at when developing an application that is heavy on writes and needs a lot of random reads. Let’s dive in and get some hands-on experience.
HBase Example
Setting up Docker
First thing as always is to get your cloudera quickstart image up and running. Whether that is via a local instance or the Digital Ocean instance. First thing we need to do is get our data where it needs to be. You should still have the movie data from the Hive tutorial. If you don’t, follow these quick steps to get the data.
For this tutorial, we will be working with some movie data. You can download the data that we are going to be using from this website here: http://www.grouplens.org/system/files/ml-1m.zip. If you want to read about the data and how it is setup, feel free to check out this file: http://files.grouplens.org/papers/ml-1m-README.txt. In the ml-1m.zip file, you should see three files inside: movies.dat, ratings.dat, users.dat.
Let’s get that into our Cloudera Quickstart Docker image and load them into HBase.
If you aren’t using the Digital Ocean version of the tutorial, you can volume mount the directory that you saved your data into your quickstart docker image by running this command:
docker run --hostname=quickstart.cloudera --privileged=true -t -i -p8888:8888 -v /place/you/saved:/inside/your/container cloudera/quickstart:latest /usr/bin/docker-quickstart bash
Once you have the Digital Ocean instance spun up and resized, let’s run some commands to all get on the same page.
docker-machine scp -r ./Downloads/ml-1m/ root@docker-sandbox:/tmp
This command will get the data into your /tmp folder on the Digital Ocean server.
docker-machine ssh docker-sandbox
docker run --hostname=quickstart.cloudera --privileged=true -t -i -p8888:8888 -v /tmp:/dataset cloudera/quickstart:latest /usr/bin/docker-quickstart bash
Let’s put the data into HDFS quickly with some data prep.
[root@quickstart ml-1m]# hdfs dfs -mkdir -p /moviedata/ratings
[root@quickstart ml-1m]# hdfs dfs -mkdir -p /moviedata/movies
[root@quickstart ml-1m]# hdfs dfs -mkdir -p /moviedata/users
[root@quickstart /]# sed 's/::/#/g' /dataset/ml-1m/movies.dat >movies.t
[root@quickstart /]# sed 's/::/#/g' /dataset/ml-1m/ratings.dat > ratings.t
[root@quickstart /]# sed 's/::/#/g' /dataset/ml-1m/users.dat > users.t
[root@quickstart /]# hdfs dfs -put movies.t /moviedata/movies
[root@quickstart /]# hdfs dfs -put ratings.t /moviedata/ratings
[root@quickstart /]# hdfs dfs -put users.t /moviedata/users
Awesome! Let's put aside this data for now, and just get some practice interacting with HBase. We will come back to this data that we just put into HDFS shortly.
HBase Shell Examples
First thing's first. Let’s jump into the HBase shell by typing:
hbase shell 
Let’s create a table now.
create 'tabletest', 'columnfamily1'
0 row(s) in 1.5990 seconds
 
=> Hbase::Table - tabletest
In the last command, we specify the create command with tabletest as the name of our table with columnfamily1 as a column family name — the command is create '<<table name>>', '<<column family>>'.
Use the list command to print out some information about your table:
list 'tabletest'
hbase(main):002:0> list 'tabletest'
TABLE                                                                                                                   
tabletest                                                                                                               
1 row(s) in 0.0270 seconds
=> ["tabletest"]
Naturally we should put some data into this table to get a real feel for it. Let’s just throw some test data into it.
hbase(main):004:0> put 'tabletest', 'firstrow', 'columnfamily1','testValue1'
0 row(s) in 0.0130 seconds
hbase(main):005:0> put 'tabletest', 'secondrow', 'columnfamily1','testValue2'
0 row(s) in 0.0150 seconds
hbase(main):006:0> put 'tabletest', 'thirdrow', 'columnfamily1','testValue3'
0 row(s) in 0.0060 seconds
This format looks a little different.
The first part put tabletest is just telling HBase which table we want to put the data into.
firstrow denotes where in the table we want to put the data. Remember random read and writes, so we have to specify exactly where to store the data.
columnfamily1 refers to which column family we are going to be putting data into
Finally we put the value of testvalue.
It's important to note here that HBase doesn't support multiple columns in a single statement. So if you had a table with a column family with two columns under it, the put statement to add to both columns would look like the following:
PUT 'tableName', 'columnFamily:column1', 'data'
PUT 'tableName', 'columnFamily:column2', 'data'
In other words, the : is used to specify the column names.
Let’s query this data and see if it what we expected.
hbase(main):007:0> scan 'tabletest'
ROW            COLUMN+CELL                                                                             
firstrow       column=columnfamily1:, timestamp=1518293802613, value=testValue1
secondrow      column=columnfamily1:, timestamp=1518293817887, value=testValue2
thirdrow       column=columnfamily1:, timestamp=1518293828203, value=testValue3
3 row(s) in 0.0610 seconds
Perfect. Notice that when we do a scan it brings up all values in the table. It shows us the ROW and COLUMN and CELL. Notice we just put one value into the column family but there are now two. The timestamp just magically appeared. That is because every row has a timestamp.
Let’s try and get just one row of the table.
get 'tabletest', 'thirdrow'
COLUMN               CELL                                                                                    
columnfamily1:       timestamp=1518293828203, value=testValue3                                               
1 row(s) in 0.0260 seconds
This shows just the column and the cell.
A cell is the value that is at the intersection of a row and a column. To find a cell you need to know the column, row, and version (timestamp) to get the correct value. Pretty cool.
It shows us the column family and the cell with the value and the timestamp. The timestamp should match above because that timestamp is created when data is put into the row so it shouldn’t change unless something happens to that row.
Something a little different with HBase is the disable and enable commands. If you want to update the settings of a table or drop a table, you need to disable the table. Then when you are ready to use it again, enable the table. Let’s try it.
First let’s do a describe on the table.
describe 'tabletest'
Table tabletest is ENABLED
                                                                                          
tabletest
                                                                                                     
COLUMN FAMILIES DESCRIPTION
                                                                                        
{NAME => 'columnfamily1', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER =>'ROW', REPLICATION_SCOPE => '0', VERSIONS => '1'
, COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536', IN_
MEMORY => 'false', BLOCKCACHE => 'true'}                                                                                
1 row(s) in 0.0490 seconds
Notice the table is ENABLED.
disable 'tabletest'
describe 'tabletest'
Table tabletest is DISABLED                                                                                             
tabletest                                                                                                              
COLUMN FAMILIES
{NAME => 'columnfamily1', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER =>'ROW', REPLICATION_SCOPE => '0', VERSIONS => '1'
, COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536', IN_
MEMORY => 'false', BLOCKCACHE => 'true'}
1 row(s) in 0.0580 seconds
It’s now disabled. Let’s try and put some data into it and see what happens.
put 'tabletest', 'fourthrow', 'columnfamily1', 'testValue4'
ERROR: Failed 1 action: NotServingRegionException: 1 time,
Got an error. Let’s enable that table and try and put data into it.
hbase(main):013:0> enable 'tabletest'
0 row(s) in 1.2850 seconds
 
hbase(main):014:0> put 'tabletest', 'fourthrow', 'columnfamily1','testValue4'
0 row(s) in 0.0160 seconds
 
hbase(main):015:0> get 'tabletest', 'fourthrow'
COLUMN             CELL                                                                                    
columnfamily1:     timestamp=1518295072544, value=testValue4                                               
1 row(s) in 0.0240 seconds
It worked. Awesome job!! Let’s try and drop the table.
drop 'tabletest'
ERROR: Table tabletest is enabled. Disable it first.
The table is enabled so it won’t let us drop it. Perfect. Go ahead and drop that table on your own. Once you get done with that, go ahead and type quit and press enter and it’ll get you back to the main command line.
Loading Data From HDFS into HBase using Pig
Let’s get back to that data that we put into HDFS and let’s put it into HBase using Pig. First thing that we need to do is go back into the HBase shell and make three tables.
create 'users', 'userdata'
0 row(s) in 1.4860 seconds
=> Hbase::Table - users
create 'ratings', 'ratingsdata'
0 row(s) in 1.5700 seconds
=> Hbase::Table - ratings
create 'movies', 'moviedata'
0 row(s) in 1.5380 seconds
=> Hbase::Table - movies
Great! Let’s create a script to load the data using pig. Copy the following code into a file called loadHbase.pig.
movies = LOAD '/moviedata/movies/movies.t' USING PigStorage(',') AS (movieid:int, title:chararray, genres:chararray);
 
STORE movies INTO 'hbase://movies' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage('moviedata:title moviedata:genres');
 
ratings = LOAD '/moviedata/ratings/ratings.t' USING PigStorage(',') AS (userid:int, movieid:int, rating:int, tstamp:chararray);
 
STORE ratings INTO 'hbase://ratings' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage('ratingsdata:movieid ratingsdata:rating ratingsdata:tstamp');
 
users = LOAD '/moviedata/users/users.t' USING PigStorage(',') AS (userid:int, gender:chararray, age:int, occupation:int, zipcode:chararray);
 
STORE users INTO 'hbase://users' USING org.apache.pig.backend.hadoop.hbase.HBaseStorage('userdata:gender userdata:age userdata:occupation userdata:zipcode');
We are loading some data from HDFS and using a special class (hbase://) to store data into HBase for each of the files. The command to run this script is a little different because we need to add some jars (Java ARchives) to the classpath of Pig that contain the special class that we used to store the data into HBase. Run the following command and watch the magic happen.
PIG_CLASSPATH=/usr/lib/hbase/hbase-*:/usr/lib/zookeeper/zookeeper-3.4.5-cdh5.7.0.jar pig loadHBase.pig
 
HadoopVersion        PigVersion        UserId        StartedAt        FinishedAt        Features
2.6.0-cdh5.7.0        0.12.0-cdh5.7.0        root        2018-02-11 20:25:07        2018-02-11 20:28:00        UNKNOWN
 
Success!
 
Job Stats (time in seconds):
JobId        Maps        Reduces        MaxMapTime        MinMapTIme        AvgMapTime        MedianMapTime        MaxReduceTime        MinReduceTime        AvgReduceTime        MedianReducetime        Alias        Feature        Outputs
job_1518379779375_0001        1        0        80        80        80        80        n/a        n/a        n/a        n/a        ratings        MAP_ONLY        hbase://ratings,
job_1518379779375_0002        1        0        21        21        21        21        n/a        n/a        n/a        n/a        movies        MAP_ONLY        hbase://movies,
job_1518379779375_0003        1        0        20        20        20        20        n/a        n/a        n/a        n/a        users        MAP_ONLY        hbase://users,
 
Input(s):
Successfully read 1000209 records (21593882 bytes) from: "/moviedata/ratings/ratings.t"
Successfully read 3883 records (163918 bytes) from: "/moviedata/movies/movies.t"
Successfully read 6040 records (110582 bytes) from: "/moviedata/users/users.t"
 
Output(s):
Successfully stored 1000209 records in: "hbase://ratings"
Successfully stored 3883 records in: "hbase://movies"
Successfully stored 6040 records in: "hbase://users"
 
Counters:
Total records written : 1010132
Total bytes written : 0
Spillable Memory Manager spill count : 0
Total bags proactively spilled: 0
Total records proactively spilled: 0
 
Job DAG:
job_1518379779375_0001
job_1518379779375_0002
job_1518379779375_0003
Awesome. The data should now be loaded into HBase. Let’s jump into the HBase shell and do some querying of the data.
scan 'movie
 988       column=moviedata:title, timestamp=1518381804300, value=Grace of My Heart (1996)                                                                                                                        
 989       column=moviedata:genres, timestamp=1518381804300, value=Drama                                                                                                                          
 989       column=moviedata:title, timestamp=1518381804300, value=Schlafes Bruder (Brother of Sleep) (1995)                                                                                                      
 99        column=moviedata:genres, timestamp=1518381803685, value=Documentary                                                                                                                                    
 99        column=moviedata:title, timestamp=1518381803685, value=Heidi Fleiss: Hollywood Madam (1995)                                                                                                            
 990       column=moviedata:genres, timestamp=1518381804300, value=Action|Adventure|Thriller                                                                                                                      
 990       column=moviedata:title, timestamp=1518381804300, value=Maximum Risk (1996)                                                                                                                            
 991       column=moviedata:genres, timestamp=1518381804300, value=Drama|War                                                                                                                                      
 991       column=moviedata:title, timestamp=1518381804300, value=Michael Collins (1996)                                                                                                                          
 992       column=moviedata:genres, timestamp=1518381804301, value= The (1996)                                                                                                                                    
 992       column=moviedata:title, timestamp=1518381804301, value=Rich Man's Wife                                                                                                                                
 993       column=moviedata:genres, timestamp=1518381804301, value=Drama                                                                                                                                          
 993       column=moviedata:title, timestamp=1518381804301, value=Infinity (1996)                                                                                                                                
 994       column=moviedata:genres, timestamp=1518381804301, value=Drama                                                                                                                                          
 994       column=moviedata:title, timestamp=1518381804301, value=Big Night (1996)                                                                                                                                
 996       column=moviedata:genres, timestamp=1518381804301, value=Action|Drama|Western                                                                                                                          
 996       column=moviedata:title, timestamp=1518381804301, value=Last Man Standing (1996)                                                                                                                        
 997       column=moviedata:genres, timestamp=1518381804301, value=Drama|Thriller                                                                                                                                
 997       column=moviedata:title, timestamp=1518381804301, value=Caught (1996)                                                                                                                                  
 998       column=moviedata:genres, timestamp=1518381804302, value=Action|Crime                                                                                                                                  
 998       column=moviedata:title, timestamp=1518381804302, value=Set It Off (1996)                                                                                                                             
 999       column=moviedata:genres, timestamp=1518381804302, value=Crime                                                                                
 999       column=moviedata:title, timestamp=1518381804302, value=2 Days in the Valley (1996)                                                                                                                    
3883 row(s) in 14.4740 seconds
There are a lot of rows there and scan will show you them all. I just put the last couple. Great job! You’ve successfully loaded some data into HBase using Pig.
Let’s do a quick query to see how the data is setup in HBase.
get 'movies', 77
Are
COLUMN                 CELL                                                                                                                                                                                                     
moviedata:genres       timestamp=1518381803660, value=Documentary                                                                                                                                                               
moviedata:title        timestamp=1518381803660, value=Nico Icon (1995)                                                                                                                                                          
2 row(s) in 0.1390 seconds