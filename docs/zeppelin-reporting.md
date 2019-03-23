## Data Reporting with Apache Zeppelin

**Objective**: In this lab you will be introduced to Zeppelin and teach you to visualize data using Zeppelin.

**Exercise directory**: `~/data`

Zeppelin provides a powerful web-based notebook platform for data analysis and discovery.
Behind the scenes it supports Spark distributed contexts as well as other language bindings on top of Spark.

In this lab we will be using Zeppelin to run SQL queries on our geolocation, trucks, and
riskfactor data that we’ve collected earlier and visualize the result through graphs and charts.


----

### Navigate to Zeppelin Notebook

Open Zeppelin interface using browser URL: http://master1.hadoop.com.com:9995/

![welcome-to-zeppelin-800x457](https://user-images.githubusercontent.com/558905/54872184-87c86080-4d96-11e9-9aca-2ccedb5df409.jpg)

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Click on a Notebook tab at the top left and select Create new note. 

Name your notebook `Driver Risk Factor`:

![create-new-notebook-800x460](https://user-images.githubusercontent.com/558905/54872175-872fca00-4d96-11e9-9426-a42f46a7b94c.jpg)

### Download the Data

If you had trouble completing the previous tutorial or lost the risk factor data click here to download it and upload it to HDFS under `/tmp/data/`:

![save-risk-factor-800x444](https://user-images.githubusercontent.com/558905/54872182-87c86080-4d96-11e9-9fb4-b4a03b351364.jpg)

### Execute a Hive Query

In the previous Spark tutorial you already created a table finalresults or riskfactor which gives the risk factor associated with every driver. We will use the data we generated in this table to visualize which drivers have the highest risk factor. We will use the jdbc Hive interpreter to write queries in Zeppelin.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2.  Copy and paste the code below into your Zeppelin note:

```
%spark2
val hiveContext = new org.apache.spark.sql.SparkSession.Builder().getOrCreate()
val riskFactorDataFrame = spark.read.format("csv").option("header", "true").load("hdfs:///tmp/data/riskfactor.csv")
riskFactorDataFrame.createOrReplaceTempView("riskfactor")
hiveContext.sql("SELECT * FROM riskfactor LIMIT 15").show()
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Click the play button next to “ready” or “finished” to run the query in the Zeppelin notebook.

Alternative way to run query is “shift+enter.”

Initially, the query will produce the data in tabular format as shown in the screenshot.

![output_riskfactor_zeppelin_lab6](https://user-images.githubusercontent.com/558905/54872180-87c86080-4d96-11e9-86b0-5b0a51241bdb.png)

### Build Charts using Zeppelin

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Iterate through each of the tabs that appear underneath the query.

Each one will display a different type of chart depending on the data that is returned in the query.

![charts_tab_jdbc_lab6](https://user-images.githubusercontent.com/558905/54872173-872fca00-4d96-11e9-8797-396586f76fee.png)

1. After clicking on a chart, we can view extra advanced settings to tailor the view of the data we want.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Click advanced:

![bar_graph_zeppelin_lab6-800x200](https://user-images.githubusercontent.com/558905/54872172-872fca00-4d96-11e9-9bd0-fe7c450e9859.png)

2. Click settings to open the advanced chart features.

3. To make a chart with riskfactor.driverid and riskfactor.riskfactor SUM, drag the table relations into the boxes as shown in the image below:

![fields_set_keys_values_chart_lab6-800x179](https://user-images.githubusercontent.com/558905/54872177-872fca00-4d96-11e9-8120-af92d33160c5.png)

4. You should now see an image like the one below:

![driverid_riskfactor_chart_lab6-800x338](https://user-images.githubusercontent.com/558905/54872176-872fca00-4d96-11e9-81c0-0c8f3cab0b9f.png)

5. If you hover on the peaks, each will give the driverid and riskfactor:

![hover_over_peaks_lab6-800x311](https://user-images.githubusercontent.com/558905/54872178-872fca00-4d96-11e9-961b-317688ebddab.png)

6. Try experimenting with the different types of charts as well as dragging and
dropping the different table fields to see what kind of results you can obtain.

7. Let’ try a different query to find which cities and states contain the drivers with the highest risk factors:

%sql
SELECT a.driverid, a.riskfactor, b.city, b.state
FROM riskfactor a, geolocation b where a.driverid=b.driverid

![queryFor_cities_states_highest_driver_riskfactor-800x294](https://user-images.githubusercontent.com/558905/54872181-87c86080-4d96-11e9-8a04-ec5033469e8b.png)

8. After changing a few of the settings we can figure out which of the cities have the high risk factors.
Try changing the chart settings by clicking the scatterplot icon. Then make sure that the keys a.driverid
is within the xAxis field, a.riskfactor is in the yAxis field, and b.city is in the group field.
The chart should look similar to the following:

![visualize_cities_highest_driver_riskfactor_lab6-800x285](https://user-images.githubusercontent.com/558905/54872183-87c86080-4d96-11e9-9c6d-a3b6d738a9c0.png)

You can hover over the highest point to determine which driver has the highest risk factor and in which cities.

### Results

Great, now we know how to query and visualize data using Apache Zeppelin. We can leverage Zeppelin—along with our newly gained knowledge of Hive and Spark—to solve real world problems in new creative ways.

<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>