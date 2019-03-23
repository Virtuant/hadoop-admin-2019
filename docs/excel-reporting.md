## Data Reporting with Excel

**Objective**: In this lab you will be introduced to Zeppelin and teach you to visualize data using Zeppelin.

**Exercise directory**: `~/data`

In this lab, using Microsoft Excel Windows and Power View, we’ll visualize data from previous sections of this tutorial. You may use other Business Intelligence (BI) tools of your choice.

----

![excel-choose-columns](https://user-images.githubusercontent.com/558905/54872365-500ee800-4d99-11e9-8f60-85ee011ea9aa.png)
![excel-choose-data-source](https://user-images.githubusercontent.com/558905/54872366-500ee800-4d99-11e9-80ea-b69733bc1207.png)
![excel-filter-data](https://user-images.githubusercontent.com/558905/54872367-50a77e80-4d99-11e9-8df2-271ae5c16905.png)
![excel-finish](https://user-images.githubusercontent.com/558905/54872368-50a77e80-4d99-11e9-8150-9527417a6b35.png)
![excel-import-data](https://user-images.githubusercontent.com/558905/54872369-50a77e80-4d99-11e9-89a5-52b6bc231138.png)
![excel-new-query](https://user-images.githubusercontent.com/558905/54872370-50a77e80-4d99-11e9-85b5-df1b231e1812.png)
![excel-open-query](https://user-images.githubusercontent.com/558905/54872371-50a77e80-4d99-11e9-8864-71eef78c33be.jpg)
![excel-power-view-icon](https://user-images.githubusercontent.com/558905/54872372-50a77e80-4d99-11e9-83b2-542fd302236c.png)
![excel-query](https://user-images.githubusercontent.com/558905/54872373-50a77e80-4d99-11e9-91a2-b50d2fdaba57.png)
![excel-query-sample](https://user-images.githubusercontent.com/558905/54872374-50a77e80-4d99-11e9-864f-36040474fb19.png)
![excel-select-map](https://user-images.githubusercontent.com/558905/54872375-50a77e80-4d99-11e9-8991-4f73bd740d90.png)
![excel-select-stacked-column](https://user-images.githubusercontent.com/558905/54872376-51401500-4d99-11e9-86cf-dec185f8877f.png)
![excel-sort-order](https://user-images.githubusercontent.com/558905/54872377-51401500-4d99-11e9-9764-e7b3da717d2b.png)
![Lab5_7-800x353](https://user-images.githubusercontent.com/558905/54872378-51401500-4d99-11e9-97a9-158932ca53bf.jpg)
![Lab5_9-800x566](https://user-images.githubusercontent.com/558905/54872379-51401500-4d99-11e9-8d46-a073a2cbb2b8.jpg)
![Lab5_15-800x564](https://user-images.githubusercontent.com/558905/54872380-51401500-4d99-11e9-95da-870e3f28f671.jpg)



### Access Data in Microsoft Excel

Let’s bring in data from table avg_mileage. We created this table in the Hive – Data ETL section.

1. Open a new blank workbook.

2. Select Data > From Other Sources > From Microsoft Query

excel-open-query

3. On the Choose Data Source pop-up, select the Hortonworks ODBC data source you installed previously, then click OK.

excel-choose-data-source

4. In the Query Wizard, select the avg_mileage table and add columns to the query, then click Next.

excel-choose-columns

5. For the following Query Wizard forms, accept the defaults and click Next.

excel-filter-data

excel-sort-order

On this last form, click Finish.

excel-finish

6. Excel will send a data request to Hive. When data is returned, it will ask you where to import the table. Accept the default location to import the table: current workbook, current worksheet, in cell $A$1 – click OK.

excel-import-data

We have successfully imported table avg_mileage into Excel. Now we are ready to do some visualization.

Lab5_7
Visualize Data with Microsoft Excel

We will use Power View to visulaize our data.

1. click on excel-power-view-icon. You created this icon as part of the Power View prerequisite. The default is to create Power View sheet, click OK.

2. We will create a column chart to visually describe the average miles per gallon for each truck. Select DESIGN > Column Chart > Stacked Column. You will need to stretch the chart by dragging the lower right of the chart to the full pane. You can control the amount of data you see by filtering on avgmpg and/or truckid.

excel-select-stacked-column

Lab5_9

Moving on to our next visual…

We’ll be using data from table geolocation. We created this table in the Hive – Data ETL section. We will create a geographical map describing the location of each truck. We’ll use the following query to gather driverid, city, and state from the table.

SELECT driverid, city, state FROM geolocation;

1. Select Data > New Query > From Other Sources > From ODBC

excel-new-query

2. Fill out From ODBC form as follows:

    Data source name (DSN): <data source name you created>
    Under Advanced options, SQL statement (optional), type: SELECT driverid, city, state FROM geolocation;
    press OK

excel-query

3. Excel will display a sample set of the query results. Click Load to create a new sheet and import the data.

excel-query-sample

4. click on excel-power-view-icon. You created this icon as part of the Power View prerequisite. The default is to create Power View sheet, click OK.

5. We will create a map to visually describe the location of each truck. Select DESIGN > Map. You will need to stretch the chart by dragging the lower right of the chart to the full pane.

    Make sure you have network connectivity because Power View uses Bing to do geocoding which translates city and state columns into map coordinates.

excel-select-map

    Uncheck driverid. We only want to see city and state.

The finished map looks like this.

Lab5_15

### Results

Congratulations! You are able to visualize your data using Microsoft Excel.

This tutorial has shown how HDP can store and visualize geolocation data with Microsoft Excel. There are many other Business Intelligent (BI) tools available you can use.

You can further explorer other visualization, such as plotting risk factor or miles per gallon as bar charts.

<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>