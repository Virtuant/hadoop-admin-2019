# Info to use for this lab: Some of the info from the Hbase shell lab is applicable to this lab as well:

https://hortonworks.com/hadoop-tutorial/introduction-apache-hbase-concepts-apache-phoenix-new-backup-restore-utility-hbase/

https://community.hortonworks.com/questions/40938/combine-few-hbase-shell-commands.html

## Lab: Data Access in the HBase Shell

## Objective:
To learn how to use the HBase Shell to put and get data.

### File locations:

### Successful outcome:
You Will:
Use the HBase Shell to put and get data.

### Lab Steps

Complete the setup guide


**HDFS paths:**          `/hbase/data/default` `/hbase/data/default/user`

**HBase tables**:       `user`

In this exercise you will use the HBase Shell to put and get data.

----

### Add Data to HBase Tables

1.  Invoke the HBase shell:

    ```console
    # hbase shell
    ```
    
1.  Describe the user table to remind yourself of the column family names.

1.  Get the row with row key 10000 from the user. The row should not exist.

1.  Add a row with the following characteristics:

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
    
1.  Get the row with row key 10000 from the user. The row should exist now.

1.  Do another put with the following characteristics:

    ```console
    Table name: ‘user’
    Row key: ‘10000’
    Column Family: ‘info’
    Column Descriptor: ‘age’ and value ‘2’
    ```

1.  Do a third put with the following characteristics:

    ```console
    Table name: ‘user’
    Row key: ‘10000’
    Column Family: ‘info’
    Column Descriptor: ‘age’ and value ‘3’
    ```

1.  Get the row with row key 10000 from the user. Note that the value returned for age is 3.

1.  Get all of the previous versions of the age column with the 10000 row key from the user.

1.  View the entire table, but only look at the age column:

    ```console
    hbase> scan 'user', {COLUMNS => 'info:age'}
    ```
    
### Delete Columns and Rows from HBase Tables

1.  Delete the age column descriptor from the user table with the 10000 row key.

1.  Verify that the age column descriptor has been removed.

1.  Delete the entire row from the user table with the 10000 row key.

1.  Verify that the row with row key 10000 has been removed from table user.

### Explore HBase Directories in HDFS

1.  Using another terminal, view the HBase directory structure in HDFS:

    ```console
    # hdfs dfs -ls /hbase/data/default
    ```
    
    This directory holds all of the tables for HBase and some other files used by HBase.

1.  View the directory structure of a table in HDFS:

    ```console
    # hdfs dfs -ls /hbase/data/default/user/*
    ```
    
### Result:

This directory shows how a table is broken in to regions.


