## Processing Data with Apache Pig

**Objective**: We will learn to store data files using Ambari HDFS Files View. We will implement pig latin scripts to process, analyze and manipulate data files of truck drivers statistics.

**Exercise directory**: `~/data`

**HDFS paths:** `/user/[user-name]`

Pig is a high level scripting language that is used with Hadoop. Pig excels at describing data analysis problems as data flows. Pig is complete in that you can do all the required data manipulations in Hadoop with Pig. In addition through the User Defined Functions(UDF) facility in Pig you can have Pig invoke code in many languages like JRuby, Jython and Java. Conversely you can execute Pig scripts in other languages. The result is that you can use Pig as a component to build larger and more complex applications that tackle real business problems.

A good example of a Pig application is the ETL transaction model that describes how a process will extract data from a source, transform it according to a rule set and then load it into a datastore. Pig can ingest data from files, streams or other sources using the User Defined Functions (UDF). Once it has the data it can perform select, iteration, and other transforms over the data. Again the UDF feature allows passing the data to more complex algorithms for the transform. Finally Pig can store the results into the HDFS.

Pig scripts are translated into a series of MapReduce jobs that are run on the Hadoop cluster. As part of the translation the Pig interpreter does perform optimizations to speed execution on Hadoop. We are going to write a Pig script that will do our data analysis task.

We are going to read in a truck driver statistics files. We are going to compute the sum of hours and miles logged driven by a truck driver for an year. Once we have the sum of hours and miles logged, we will extend the script to translate a driver id field into the name of the drivers by joining two different files.

----

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Download The Data</h4>

Download the driver data file from [here](https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/how-to-process-data-with-apache-hive/assets/driver_data.zip).

Once you have the file you will need to unzip the file into a directory. We will be uploading two csv files – `drivers.csv ` and `timesheet.csv`.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Upload the data files</h4>

We start by selecting the HDFS Files view from the off-canvas menu at the top. The HDFS Files view allows us to view the Hortonworks Data Platform (HDP) file store. This is separate from the local file system.

![files-view](https://user-images.githubusercontent.com/558905/54872452-aa5c7880-4d9a-11e9-84be-e9ecb79b56de.jpg)

Navigate to `/user/student` and click on the Upload button to select the files we want to upload:

![upload-button-800x241](https://user-images.githubusercontent.com/558905/54872468-ab8da580-4d9a-11e9-824d-1f9799095c86.jpg)

Click on the browse button to open a dialog box. Navigate to where you stored the `drivers.csv` file on your local disk and select `drivers.csv` and click Open. Do the same thing for `timesheet.csv`. When you are done you will see there are two new files in your directory:

![uploaded_files-800x301](https://user-images.githubusercontent.com/558905/54872469-ab8da580-4d9a-11e9-9efe-5d2cd7623074.png)

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Create a Pig Script</h4>

To get started create a new file named `sum_of_hours_miles` then use VI or `nano` to edit it:

```
touch sum_of_hours_miles
vi sum_of_hours_miles
```

The first thing we need to do is load the data. We use the load statement for this. The PigStorage function is what does the loading and we pass it a comma as the data delimiter. Our code is:

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) To enter insert mode in VI press `i` and to exit press `esc` and `:x` to save. You may also exit without saving by pressing `:q!`

```
drivers = LOAD 'drivers.csv' USING PigStorage(',');
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Filter Out Data</h4>

To filter out the first row of the data we have to add this line:

```
raw_drivers = FILTER drivers BY $0>1;
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>5. Name the Fields</h4>

The next thing we want to do is name the fields. We will use a FOREACH statement to iterate through the raw_drivers data object and GENERATE pulls out selected fields and assigns them names. The new data object we are creating is then named driver_details. Our code will now be:

```
drivers_details = FOREACH raw_drivers GENERATE $0 AS driverId, $1 AS name;
```

Perform these operations for timesheet data as well - load the timesheet data and then filter out the first row of the data to remove column headings and then use FOREACH statement to iterate each row and GENERATE to pull out selected fields and assign them names.

```
timesheet = LOAD 'timesheet.csv' USING PigStorage(',');
raw_timesheet = FILTER timesheet by $0>1;
timesheet_logged = FOREACH raw_timesheet GENERATE $0 AS driverId, $2 AS hours_logged, $3 AS miles_logged;
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>6. Now Filter The Data (all hours and miles for each driverId)</h4>

The next line of code is a GROUP statement that groups the elements in timesheet_logged by the driverId field. So the grp_logged object will then be indexed by driverId. In the next statement as we iterate through grp_logged we will go through driverId by driverId. Type in the code:

```
grp_logged = GROUP timesheet_logged by driverId;
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>7. Now Find the Sum of Hours and Miles Logged by each Driver</h4>

In the next FOREACH statement, we are going to find the sum of hours and miles logged by each driver. The code for this is:

```
sum_logged = FOREACH grp_logged GENERATE group as driverId,
SUM(timesheet_logged.hours_logged) as sum_hourslogged,
SUM(timesheet_logged.miles_logged) as sum_mileslogged;
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>8. Build a Script to join driverId, Name, Hours and Miles Logged</h4>

Now that we have the sum of hours and miles logged, we need to join this with the driver_details data object so we can pick up the name of the driver. The result will be a dataset with driverId, name, hours logged and miles logged. At the end we DUMP the data to the output.

```
join_sum_logged = JOIN sum_logged by driverId, drivers_details by driverId;
join_data = FOREACH join_sum_logged GENERATE $0 as driverId, $4 as name, $1 as hours_logged, $2 as miles_logged;
dump join_data;
```

Now let’s take a look at our script. The first thing to notice is we never really address single rows of data to the left of the equals sign and on the right we just describe what we want to do for each row. We just assume things are applied to all the rows. We also have powerful operators like GROUP and JOIN to sort rows by a key and to build new data objects.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>9. Save and Execute The Script</h4>

At this point we can save our script. Let’s execute our code by exiting and saving on VI press esc then type :x

Next submit the Pig job:

```
pig -x mr -f sum_of_hours_miles
```

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png)  `-x mr` sets the execution engine to be MapReduce.

![pig-submit-800x70](https://user-images.githubusercontent.com/558905/54872459-aaf50f00-4d9a-11e9-86bb-114c6b7001f0.jpg)

As the jobs are run we will get status boxes where we will see logs, error message, the output of our script and our code at the bottom.

![script-results-800x521](https://user-images.githubusercontent.com/558905/54872462-aaf50f00-4d9a-11e9-9bf4-e58bbf1677a7.jpg)

If you scroll up to the program output you can see the log file of your jobs. We should always check the Logs to check if your script was executed correctly.

![script-logs-800x708](https://user-images.githubusercontent.com/558905/54872461-aaf50f00-4d9a-11e9-91ce-3c04ae5335a5.jpg)

### Code Recap

* So we have created a simple Pig script that reads in some comma separated data.
* Once we have that set of records in Pig we pull out the driverId, hours logged and miles logged fields from each row.
* We then group them by driverId with one statement, GROUP.
* Then we find the sum of hours and miles logged for each driverId.
* This is finally mapped to the driver name by joining two datasets and we produce our final dataset.

As mentioned before Pig operates on data flows. We consider each group of rows together and we specify how we operate on them as a group. As the datasets get larger and/or add fields our Pig script will remain pretty much the same because it is concentrating on how we want to manipulate the data.

### Full Pig Latin Script for Exercise

```
drivers = LOAD 'drivers.csv' USING PigStorage(',');
raw_drivers = FILTER drivers BY $0>1;
drivers_details = FOREACH raw_drivers GENERATE $0 AS driverId, $1 AS name;
timesheet = LOAD 'timesheet.csv' USING PigStorage(',');
raw_timesheet = FILTER timesheet by $0>1;
timesheet_logged = FOREACH raw_timesheet GENERATE $0 AS driverId, $2 AS hours_logged, $3 AS miles_logged;
grp_logged = GROUP timesheet_logged by driverId;
sum_logged = FOREACH grp_logged GENERATE group as driverId,
SUM(timesheet_logged.hours_logged) as sum_hourslogged,
SUM(timesheet_logged.miles_logged) as sum_mileslogged;
join_sum_logged = JOIN sum_logged by driverId, drivers_details by driverId;
join_data = FOREACH join_sum_logged GENERATE $0 as driverId, $4 as name, $1 as hours_logged, $2 as miles_logged;
dump join_data;
```


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>Extra Credit</h4>

#### What is Tez?

Tez – Hindi for “speed” provides a general-purpose, highly customizable framework that creates simplifies data-processing tasks across both small scale (low-latency) and large-scale (high throughput) workloads in Hadoop. It generalizes the MapReduce paradigm to a more powerful framework by providing the ability to execute a complex DAG (directed acyclic graph) of tasks for a single job so that projects in the Apache Hadoop ecosystem such as Apache Hive, Apache Pig and Cascading can meet requirements for human-interactive response times and extreme throughput at petabyte scale (clearly MapReduce has been a key driver in achieving this).

### Run Pig Script on Tez

So let’s run the same Pig script with Tez by clicking on Execute on Tez by resubmitting the Pig Job and indicating the execution enginge to be Tez:

```
pig -x tez -f sum_of_hours_miles
```

Notice that Tez is signigicantly faster than MapReduce.

Time with MR:

![script_logs_with_tez-800x43](https://user-images.githubusercontent.com/558905/54872460-aaf50f00-4d9a-11e9-99a6-8dc63510a680.jpg)

![top-form-bg](https://user-images.githubusercontent.com/558905/54872467-ab8da580-4d9a-11e9-8595-fc54c0bd24a8.png)

Time with Tez:

![time-with-tez-800x42](https://user-images.githubusercontent.com/558905/54872466-ab8da580-4d9a-11e9-980a-1d00e8aeab98.jpg)

On our machine it took around 33 seconds with Pig using the Tez engine. That is nearly 3X faster than Pig using MapReduce even without any specific optimization in the script for Tez.
Tez definitely lives up to it’s name.

### Results



<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>
