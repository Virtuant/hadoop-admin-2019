## Interactive Query with Hive and Tez

**Objective**: We will learn to store data files using Ambari HDFS Files View. We will implement pig latin scripts to process, analyze and manipulate data files of truck drivers statistics.

**Exercise directory**: `~/data`

**HDFS paths:** `/user/[user-name]`

In this tutorial, we’ll focus on taking advantage of the improvements to Apache Hive and Apache Tez through the work completed by the community as part of the Stinger initiative, some of the features which helped make Hive be over one hundred times faster are:

* Performance improvements of Hive on Tez
* Performance improvements of Vectorized Query
* Cost-based Optimization Plans
* Multi-tenancy with HiveServer2


----

![add-new-table-800x417](https://user-images.githubusercontent.com/558905/54872546-0a075380-4d9c-11e9-972c-84084c4c6ef7.jpg)
![all_applications-800x135](https://user-images.githubusercontent.com/558905/54872547-0a075380-4d9c-11e9-9949-1606c1c72cb6.jpg)
![das-ui-splash-800x496](https://user-images.githubusercontent.com/558905/54872548-0a075380-4d9c-11e9-968d-1fceadffbf17.jpg)
![first_join_mr-800x386](https://user-images.githubusercontent.com/558905/54872549-0a075380-4d9c-11e9-9907-4bb33a8ba120.jpg)
![first_join_tez_logs-800x481](https://user-images.githubusercontent.com/558905/54872550-0a075380-4d9c-11e9-82e6-f2a717fc5e81.jpg)
![hive-exec-eng-800x350](https://user-images.githubusercontent.com/558905/54872551-0a075380-4d9c-11e9-9793-52a52ac81619.jpg)
![review-data-ddl-800x709](https://user-images.githubusercontent.com/558905/54872552-0a075380-4d9c-11e9-8173-83a22a35d1bc.jpg)
![second_join_mr-800x540](https://user-images.githubusercontent.com/558905/54872553-0a075380-4d9c-11e9-8bd2-8f62fa910fa1.jpg)
![settings_page-800x426](https://user-images.githubusercontent.com/558905/54872554-0a9fea00-4d9c-11e9-980a-eb4cdd21ed62.jpg)
![uncheck-auto-convert-800x350](https://user-images.githubusercontent.com/558905/54872555-0a9fea00-4d9c-11e9-8ed0-d26215bf3230.jpg)
![upload-files-800x719](https://user-images.githubusercontent.com/558905/54872556-0a9fea00-4d9c-11e9-9c10-dd6910d076e7.jpg)
![welcome-to-das-800x389](https://user-images.githubusercontent.com/558905/54872557-0a9fea00-4d9c-11e9-8850-69236131efff.jpg)


Download Data

Download the data from our repository:

wget https://github.com/hortonworks/data-tutorials/raw/master/tutorials/hdp/interactive-query-for-hadoop-with-apache-hive-on-apache-tez/assets/driver_data.zip
unzip driver_data.zip

Alternatively, click here to download.

We will be uploading two csv files – drivers.csv and timesheet.csv on to DAS to create tables from them.
Create Hive Tables from CSV files on DAS

DAS can be accessed by selecting the service from Sandbox Splash Page

das-ui-splash

DAS is also accessible by navigating to sandbox-hdp.hortonworks.com:30800

You will find the Data Analytics Studio UI:

welcome-to-das

Next, we will create tables based on the csv files we downloaded earlier.

1. Click on Database

2. Select the + button to add a new table

3. Click on UPLOAD TABLE

add-new-table

4. Select the Is first row header? checkbox

5. Select Upload from Local

6. Drag and drop drivers.csv and timesheet.csv onto the browser or select the files from your local directory

upload-files

7. Review the data and the DDL, once you are satisfied select create

review-data-ddl

Here is a table of the DDL for your reference:

Driver Table
Item 	Data Type
driverId 	int
name 	string
ssn 	big int
location 	string
certified 	string
wageplan 	string

Timesheet Table
driverId 	int
week 	int
hoursLogged 	int
miles_logged 	int
Speed Improvements

To experience the speed improvements of Hive on Tez, we will run some sample queries.

By default, the Hive view runs with Tez as it’s execution engine. That’s because Tez has great speed improvements over the original MapReduce execution engine. But by how much exactly are these improvements? Well let’s find out!
Configure MapReduce as Execution Engine in Hive view Settings Tab

Great, now that our tables are created and loaded with data we can begin experimenting with settings:

Navigate back to Ambari and sign in as Username/Password: raj_ops/raj_ops

Next select Hive and then CONFIGS

settings_page

Finally, use the filter to find and modify these specific configurations:
Configuration 	New Value
hive.execution.engine 	mr
hive.auto.convert.join 	false (unselect box)

hive-exec-eng

uncheck-auto-convert

Save the changes and restart all services required.
Test Query on MapReduce Engine

We are now going to test a query using MapReduce as our execution engine. Head back to DAS then execute the following query and wait for the results.

select d.*, t.hoursLogged, t.milesLogged
from drivers d join timesheet t
on d.driverId = t.driverId;

first_join_mr

This query was run using the MapReduce framework.
Configure Tez as Execution Engine in Hive Settings Tab

Now we can enable Hive on Tez execution and take advantage of Directed Acyclic Graph (DAG) execution representing the query instead of multiple stages of MapReduce program which involved a lot of synchronization, barriers and IO overheads. This is improved in Tez, by writing intermediate data set into memory instead of hard disk.

Great, now that our tables are created and loaded with data we can begin experimenting with settings:

Navigate back to Ambari and sign in as Username/Password: raj_ops/raj_ops

Next select Hive and then CONFIGS

settings_page

Finally, use the filter to find and modify these specific configurations:
Configuration 	New Value
hive.execution.engine 	tez
hive.auto.convert.join 	true (select box)

Save the changes and restart all services required.
Test Query on Tez Engine

Run the same query as we had run earlier to see the speed improvements with Tez.

select d.*, t.hours_logged, t.miles_logged
from drivers d join timesheet t
on d.driverId = t.driverId;

first_join_mr1

Take a look at the Visual Explain to visually see the execution plan.

first_join_tez_logs

Notice that the results will have appeared much quicker while having the execution engine set to Tez. This is currently the default for all Hive queries.

Congratulations! You have successfully run your Hive on Tez Job.
Execute Query as MapReduce Then Tez Engine

Now let’s try a new query to work with

SELECT d.driverId, d.name, t.total_hours, t.total_miles from drivers d
JOIN (SELECT driverId, sum(hoursLogged)total_hours, sum(milesLogged)total_miles FROM timesheet GROUP BY driverId ) t
ON (d.driverId = t.driverId);

Try executing the query first on MapReduce execution engine, then on Tez. You should notice a considerable gap in execution time.
Here is the result.

second_join_mr

To experience this further, you could use your own dataset, upload to your HDP Sandbox using steps above and execute with and without Tez to compare the difference.
Track Hive on Tez Jobs

You can track your Hive on Tez jobs in HDP Sandbox Web UI as well by navigating to http://sandbox-hdp.hortonworks.com:8088/ui2, here you can observe the running state of the queries we just experimented with, as well as the engine used in the background.

all_applications

You can click on your job and see further details.

### Results

You learned how to create Hive tables from .csv files using DAS. We also experimented with MapReduce and Tez to observe the speed improvements of Hive on Tez.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>