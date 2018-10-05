## Lab: Importing Data into HBase from MySQL

**Objective**: In this exercise you will import data from MySQL into HBase.

**Exercise directory**: `~/data/movielens` `~/data/hbase`

**MySQL database**:     `movielens`

**MySQL tables**:       `movie, movierating, genre, moviegenre, occupation, user`

**HBase tables**:       `movie, user, ratings`

**Import scripts**:     `importMovieRatings.sh`  

In this exercise you will import data from MySQL into HBase. Your company was using MySQL and started hitting scale issues as the traffic grew. You have been tasked with migrating the data from MySQL to HBase.

----

### Examine MySQL Tables

Change directories to the exercise directory containing the data import scripts and code. If the data has not been imported to MySQL go ahead and do it now. Go to the `data/movielens` directory and execute:

```console
    mysql -u root -p < movielens.sql
    Enter password:
```
    
The password is `hadoop`.

Let’s first look at the data you will import shortly. The data is in MySQL tables. Issue the following commands to enter the MySQL shell. You will use tables that are in the movielens database:

```sql
    mysql> use movielens;  
    mysql> show tables;
```
    
Issue the following command to count the number of records in the movie Remember this number for later to verify that all MySQL records were successfully loaded into the HBase movie table:

```sql
    mysql> select count(*) from movie;
```
    
Next find out how many records are in the movierating table:

```sql
    mysql> select count(*) from movierating;
```
    
Finally, find movie ratings for the user with user ID 4185:

```sql
    mysql> select * from movierating where userid = 4185;
```
    
Exit the MySQL shell:

```sql
    mysql> exit;
```
    
### Create HBase Tables

Next create the HBase tables that you will load with data from the MySQL tables you looked at above. First, create a new HBase table called `movie` by issuing:

```console
    hbase shell
    > create 'movie', {NAME => 'info', VERSIONS => 3}, {NAME => 'media', VERSIONS => 3}
```
    
This creates a new table in HBase named `movie` that does not contain any rows. You will use another command to move the rows from MySQL into HBase.

Create another new HBase table called `user` by issuing:

```console
    create 'user', {NAME => 'info', VERSIONS => 3}, {NAME => 'ratings', VERSIONS => 3}
```
    
This creates another new table in HBase that does not contain any rows. You will use a different command to move the rows from MySQL into HBase. 
    
Now exit the shell. 

### Load MySQL Data into HBase Tables

In the following steps, you will run scripts that will compile and run a Java program to import data with the HBase API. The program will connect to MySQL, process the data, and use the HBase API to put the data in HBase.

Import the movie data from MySQL by running:

```console
    data/hbase/importMovieRatings.sh
```
    
When complete, go back into the shell and list the tables, and query the `movierating` table:
    
```console
    hbase> count ‘movierating’, INTERVAL => 100000
    hbase> count ‘movierating’, CACHE => 1000
    hbase> count ‘movierating’, INTERVAL => 10, CACHE => 1000
```
    
and scan the table:
    
```console
    hbase> scan 'movierating'
```
    
and `<CTRL>-C` to break out.

### Summary


