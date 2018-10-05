## Lab: Data Access in the HBase Shell

**HBase tables**:       `user`

In this exercise you will use the HBase Shell to put and get data. 

----

### Add Data to HBase Tables

Invoke the HBase shell.
    
Describe the user table to remind yourself of the column family names.

Get the row with row key 10000 from the user. The row should not exist.

Add a row with the following characteristics:

    ```console
    Table name: 'user'
    Row key: '10000'
    Column Family: 'info'
    Column Descriptor: 'age' and value '1'
    Column Descriptor: 'gender' and value 'F'
    Column Descriptor: 'occupationname' and value 'Toddling'
    Column Descriptor: 'zip' and value '90210'
    ```

    Hint:
    
    ```console
    hbase> put 'user', '10000', 'info:age', '1'
    ```
    
Get the row with row key 10000 from the user. The row should exist now.

Do another put with the following characteristics:

    ```console
    Table name: ‘user’
    Row key: ‘10000’
    Column Family: ‘info’
    Column Descriptor: ‘age’ and value ‘2’
    ```

Do a third put with the following characteristics:

    ```console
    Table name: ‘user’
    Row key: ‘10000’
    Column Family: ‘info’
    Column Descriptor: ‘age’ and value ‘3’
    ```

Get the row with row key 10000 from the user. Note that the value returned for age is 3.

Get all of the previous versions of the age column with the 10000 row key from the user.

View the entire table, but only look at the age column:

    ```console
    hbase> scan 'user', {COLUMNS => 'info:age'}
    ```
    
### Delete Columns and Rows from HBase Tables

Delete the age column descriptor from the user table with the 10000 row key.

Verify that the age column descriptor has been removed.

Delete the entire row from the user table with the 10000 row key.

Verify that the row with row key 10000 has been removed from table user.

### Explore HBase Directories in HDFS

Using another terminal, view the HBase directory structure in HDFS:

    ```console
    hdfs dfs -ls /hbase/data/default
    ```
    
This directory holds all of the tables for HBase and some other files used by HBase.

View the directory structure of a table in HDFS:

    ```console
    dfs dfs -ls /hbase/data/default/user/*
    ```
    
### Result:

This directory shows how a table is broken in to regions.
