## Using Spark Transformations and Actions

### About This Lab

**Objective:** To use Spark and RDD transformations and actions

**Successful outcome:** Write a Spark application that computes the most frequent visitors to the President at the White House

**File Locations:** `~/labs`

---
### Steps



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1 . Create a New Notebook</h2>

1\.  Go to the IPython Notebook/Jupyter dashboard page

2\.  In the labs folder, create a new notebook as before and rename it `Spark_Lab`. 
  


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Import SparkContext</h2>


1\.  In the first cell, import the `SparkContext` module and create a `SparkContext` ("sc") object:

![using-spark-transformations-and-actions-1](https://user-images.githubusercontent.com/21102559/40943075-f449cb90-681d-11e8-9d5b-f87e0e58a8d0.png)

2\.  Run the cell, which may take a moment the first time it is executed.
    
    

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. Create an RDD</h2>


1\.  You are going to analyze the White House visitor data, which is in a file named `whitehouse/visits.txt`. In the second cell, add the following two lines:

![using-spark-transformations-and-actions-2](https://user-images.githubusercontent.com/21102559/40943077-f45c1f16-681d-11e8-9710-73b76cf1f574.png)


2.  Run the cell. Notice the output shows the data type of `sourcefile`, which is a `MappedRDD` object:

```
MappedRDD\[14\] at textFile at NativeMethodAccessorImpl.java:-2
```

If you get an error, make sure `whitehouse/visits.txt` is in HDFS. The source file can be found locally in the `~/ds/labs/Lab4.1` folder on your VM.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>4. Project the Desired Columns</h2>

1\.  The White House visitor data has a lot of columns.

The first (`col 0`), second (`col1`), and 20th (`Col 19`) columns are the visitor's last name, first name, and the person they visited. In a new cell, define the following project function that extracts these three columns:

```
def project(rec):
    fields = rec.split(',')
    return (", ".join(list([fields[i] for i in [0, 1]])), fields[19])
 
```

2\.  Map the data in the text file using the project function:

```
projection = sourcefile.map(lambda rec: project(rec))
```

3\.  Using the `take` action, display 10 of the results of **projection**.

4\.  Run the cell. The output should look like:

![using-spark-transformations-and-actions-3](https://user-images.githubusercontent.com/21102559/40943078-f46d1690-681d-11e8-89c5-15b8969c02c0.png)

In the output above, each visitor visited someone named "**Adams**". 

> Note In the projection RDD, notice that its records are (`key,value`) pairs, where key is the name of the visitor and value is the person visited. This  format is useful (and necessary) in the upcoming steps when you will use the reduceByKey and sortByKey transformations, which both require the data to be in a (`key,value`) format.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>5. Filter Visitors of the POTUS</h2>

1\.  Use the `filter` transformation to obtain visitors of the President:

```
potus = projection.filter(lambda rec: \"POTUS\" in rec\[1\])
```

2\.  Use the take action to display 10 of the values in `potus`. Run the cell.

The output should look like:

![using-spark-transformations-and-actions-4](https://user-images.githubusercontent.com/21102559/40943079-f47c1f46-681d-11e8-85c0-3952d221c769.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Count the Number of Visits</h2>

1\.  The potus RDD has duplicate entries for visitors who met with the `POTUS` multiple times. Define the following transformation in a new cell, which adds the number "1" to the end of each visitor's name (and also removes the `POTUS` field):

```
potus_count = potus.map(lambda rec: (rec[0], 1))
```

2\.  Use the take action to display 10 of the results. Run the cell. The output should look like:

![using-spark-transformations-and-actions-5](https://user-images.githubusercontent.com/21102559/40943081-f4930c7e-681d-11e8-9704-1e8763251528.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. Sum the Visits</h2>

1\.  Perform the following `reduceByKey` transformation, which sums up the "1's" column for each visitor:

```
counts = potus_count.reduceByKey(lambda a,b: a + b)
```

2\.  Use the take action to display 10 of the results in`counts`. Run the cell.

3\.  The output should look like:

![using-spark-transformations-and-actions-6](https://user-images.githubusercontent.com/21102559/40943082-f4a2ab0c-681d-11e8-8ea6-61b27fd53206.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>8. Sort the Results</h2>

1\.  In this step, see if you can figure out how to output the visitors in the counts transformation sorted by the most frequent visitor first. To do this, you should first reverse the ordering of the tuples so that the number of visits appears first. For example:
```
(1, u'DONOVAN, MICHAEL'), 
(3, u'PHELPS, DARREN'),
(1, u'RASSAS, TODD'),
(1, u'DUBRAY, MEGAN'),
(2, u'SCHUYLER, NICHOLAS'), 
(3, u'BELL, JAMES'),
(1, u'CALHOUN, PEGGY'), 
(1, u'LANE, LAURA'), 
(2, u'SERRANO, JOSE'), 
(2, u'TANSEY, WILLIAM')
```

2\.  Then use the `sortByKey` transformation to sort the above dataset in descending order.

3\.  Use the take action and display the 100 most frequent visitors.

4\. The correct result should look like:

```
[(16, u'PRATHER, ALAN'),
(15, u'MOTTOLA, ANNAMARIA'), 
(15, u'FRANKE, CHRISTOPHER'), 
(14, u'BOGUSLAW, ROBERT'), 
(14, u'POWERS, CHARLES'), 
(12, u'SATO, FERN'),
(12, u'FISH, DIANA'),
(12, u'WALKER, JACKIE'),
(12, u'WANG, SHENGTSUNG'), 
(12, u'HART, SARAH'),
(12, u'FETTIG, JASON'),
(11, u'DEWEY, GLENN'),
(11, u'WILSON, PETER'),
(11, u'BAILEY, JANET'),
(11, u'BOTELHO, MARCIO'), 
(11, u'WILLINGHAM, DONNA'), 
(10, u'CHUDACOFF, CLAUDIA'), 
...
```

### Result

You have written a Spark application that computes the most frequent visitors to the President at the White House. You should now be comfortable with the basics of Spark and how to use transformations and actions.

You are finished!


---
### Solution

##### Step 8:
```
counts_key = counts.map(lambda rec: (rec[1], rec[0])) 
sorted_counts = counts_key.sortByKey(ascending = False) 
sorted_counts.take(100)
```
