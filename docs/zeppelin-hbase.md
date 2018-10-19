## Lab: Zeppelin and Phoenix for HBase

Zeppelin is a browser-based notebook-driven system that enables data engineers, data analysts and data scientists to be more productive by developing, organizing, executing, and sharing data code and visualizing results without referring to the command line or needing the cluster details. 

Notebooks allow these users not only allow to execute but to interactively work with long workflows.  There are a number of notebooks available.  The [Hortonworks Gallery](https://community.hortonworks.com/repos/index.html) provides an Ambari stack definition to help users quickly set up Zeppelin-related tech on their Hadoop clusters.  

Apache Zeppelin is a new and upcoming web-based notebook which brings data exploration, visualization, sharing and collaboration features to Spark. It supports Python, Scala, Hive, SparkSQL, shell and markdown. 

This lab introduces you to the SQL over Phoenix to HBase. Check it out.

----
### Go to Zeppelin

Go to Ambari, and select the Zeppelin page. If it is not running, Start it.

<img width="1168" alt="screen shot 2018-08-15 at 12 03 57 pm" src="https://user-images.githubusercontent.com/558905/44158717-69bdb380-a083-11e8-978c-82038b4f165f.png">

Click on the Quick Links `Zeppelin UI` link.

Log in as the same user and password as Ambari.

### Create a Zeppelin Notebook

Select notebook next to the Zeppelin icon, and hit “Create new note” to create the Zeppelin Notebook. Name it something like `Visualize Weather Data with Phoenix SQL`. 

>Note: choose “Default Interpreter” as 'jdbc'. On the notebook itself you will put `%jdbc(phoenix)` as the actual JDBC driver (see below).


### Create Phoenix Table Mapping to HBase Table

We must create a Phoenix table to map to our HBase table in order to perform SQL queries against HBase. Write or Copy/Paste the following query in the Zeppelin editor.

```sql
  %jdbc(phoenix)
  CREATE TABLE IF NOT EXISTS "iot_data" (
    rowkey INTEGER NOT NULL PRIMARY KEY, 
    id INTEGER, 
    parameter CHAR(20), 
    val BIGINT, 
    device_id CHAR(5), 
    datetime DATE)
```

>Note: do subsequent SQL entries in a separate swim lane so they can be re-executed easily

For purposes of making some random test data let's create a couple of sequences:

```sql
  %jdbc(phoenix)
  CREATE SEQUENCE my_sequence MINVALUE 9999999 MAXVALUE 999999999 CYCLE;
  CREATE SEQUENCE my_sequence_2 MINVALUE 45 MAXVALUE 199 CYCLE;
```

Run a quick test to verify Phoenix table successfully mapped to the HBase table.

```sql
  %jdbc(phoenix)
  SELECT * FROM system.catalog;
```

Display the first 10 rows of the Phoenix table using Zeppelin’s Table Visualization.

```sql
  %jdbc(phoenix)
  select * from "iot_data" limit 10
```

Question: is there any data yet?

### Put in Data via SQL

```sql
  %jdbc(phoenix)
  UPSERT INTO "iot_data"
  (rowkey, id, datetime, parameter, device_id, val)
  VALUES (
    NEXT VALUE FOR my_sequence, 
    NEXT VALUE FOR my_sequence_2,
    now() - ((10 * 365) + RAND()*(5*365)),
    'TEMPERATURE',
    'SBS05',
    (50 + RAND()*200)
  )
```

Now run the 10 query again:

```sql
  %jdbc(phoenix)
  select * from "iot_data" limit 10
```

And there's your data!

Now cycle the UPSERT for about 10 times.

Now run the query again.

You can also go back to terminal to view the data in the Phoenix client as well:

```sql
  /usr/hdp/current/phoenix-client/bin/sqlline.py localhost
  ...
  0: jdbc:phoenix:localhost> select * from "iot_data" limit 5;
  +-----------+-----+--------------+------+------------+--------------------------+
  |  ROWKEY   | ID  |  PARAMETER   | VAL  | DEVICE_ID  |         DATETIME         |
  +-----------+-----+--------------+------+------------+--------------------------+
  | 10000000  | 46  | TEMPERATURE  | 164  | SBS05      | 2017-07-09 18:06:40.161  |
  | 10000001  | 47  | TEMPERATURE  | 89   | SBS05      | 2017-02-02 18:18:00.416  |
  | 10000002  | 48  | TEMPERATURE  | 195  | SBS05      | 2017-04-29 18:18:23.984  |
  | 10000003  | 49  | TEMPERATURE  | 84   | SBS05      | 2017-04-01 18:19:08.923  |
  | 10000004  | 50  | TEMPERATURE  | 65   | SBS05      | 2017-12-24 18:19:23.048  |
  +-----------+-----+--------------+------+------------+--------------------------+
  5 rows selected (0.069 seconds)
  0: jdbc:phoenix:localhost> 
```

### Zeppelin’s Table Visualization

1. Temperature Over Time

Enter this query:

```sql
  %jdbc(phoenix)
  select "ID" AS ID, "VAL" AS TEMPERATURE from "iot_data"
```

And now look at it in Visual mode:

<img width="1376" alt="screen shot 2018-08-15 at 2 31 17 pm" src="https://user-images.githubusercontent.com/558905/44165909-fbcfb700-a097-11e8-96a9-3d1e0e4565e1.png">

2. Now if you have time, do the same for Humidity, Pressure, or another measure.

Get the picture?

### Results

Congratulations, now you know how to write Phoenix SQL queries against an HBase table. You performed Phoenix SQL like queries against HBase to monitor temperature, humidity and pressure over time. You also know how to use the Phoenix Interpreter integrated with Zeppelin to visualize the data associated with our weather sensor. 

Feel free to further explore the different Zeppelin Interpreters and other visualizations for your IoT data analysis journey.

This is part of a challeging exercise hooking up a [Raspberry Pi to HBase and Phoenix](https://hortonworks.com/tutorial/analyze-iot-weather-station-data-via-connected-data-architecture) to get real-time data. If you're brave, try it out!
