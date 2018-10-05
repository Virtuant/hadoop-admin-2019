## Using Spark MLib

### About This Lab

**Objective:** To use Spark MLlib to run data science algorithms on a Hadoop cluster

**Successful outcome:** Follow along with the steps below and be sure to ask question if needed. And have fun!

**File locations:** `~/labs`

---
### Steps


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. Put the Data in HDFS</h2>

1\.  View the contents of the `test.data` file:
```
more test.data
```
Output:
```
1,1,5.0
1,2,1.0
1,3,5.0
1,4,1.0
2,1,5.0
2,2,1.0
2,3,5.0
2,4,1.0
3,1,1.0
3,2,5.0
3,3,1.0
3,4,5.0
4,1,1.0
4,2,5.0
4,3,1.0
4,4,5.0
```

3\.  Notice the data consists of points along with a rating. For example, `(1,1)` has a rating of 5.0.

4\.  Put the `test.data` file in HDFS:

```
hadoop fs -put test.data
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Create a New Notebook</h2>

1\.  Go to the IPython Notebook/Jupyter dashboard page.

2\.  In the `labs` folder, create a new notebook and name it `Spark_ALS_Lab`.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Define the Imports</h2>

1\.  In the first cell, add the following imports:

```
from pyspark import SparkContext, SparkConf
conf = SparkConf().set('spark.executor.instances',1). \
    set("spark.executor.memory", "4g").set("AppName", "myapp") 
from pyspark.mllib.recommendation import ALS
from pyspark.mllib.recommendation import Rating
from numpy import array
sc = SparkContext(master="yarn-client", conf=conf)
```

2\.  Run the cell. 


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Load the Data</h2>


1\.  In a new cell, create an RDD for the `test.data` file:

```
data = sc.textFile("test.data")
```

2\.  Transform the `data` RDD by converting each entry into an array.

(Make sure the following line of code appears on a single line in your cell):

```
ratings = data.map(lambda line: Rating(*array([float(x) for x in line.split(',')])))
```

3\.  Use the `take` action to display 10 of the values:

```
ratings.take(10)
```

4\.  Run the cell.

The output should look like:

```
[Rating(user=1, product=1, rating=5.0), 
Rating(user=1, product=2, rating=1.0), 
Rating(user=1, product=3, rating=5.0), 
Rating(user=1, product=4, rating=1.0), 
Rating(user=2, product=1, rating=5.0), 
Rating(user=2, product=2, rating=1.0), 
Rating(user=2, product=3, rating=5.0), 
Rating(user=2, product=4, rating=1.0), 
Rating(user=3, product=1, rating=1.0), 
Rating(user=3, product=2, rating=5.0)]
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>5. Train an ALS Model</h2>

1\.  Using the `ratings` data, train an ALS recommender:

```
model = ALS.train(ratings, 10, 20)
```

2\.  Run the cell to build the model. 


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>6. Test the Model</h2>

1\.  In this step, you will test the model by computing the mean-squared error using the same training data as the test data. Start by creating a new RDD named `testdata`:

```
testdata = ratings.map(lambda p: (int(p\[0\]), int(p\[1\])))
```

2\.  Use the `predictAll` function of ALS to predict all the points in `testdata`:

```
predictions = model.predictAll(testdata).map(lambda r: ((r\[0\], r\[1\]), r\[2\]))
```

3\.  View the predictions:

```
predictions.collect()
```

4\.  Run the cell, which will take a minute.

5\.  The output should look like:

![using-spark-mlib-1](https://user-images.githubusercontent.com/21102559/40943055-e6ebe956-681d-11e8-8534-fe8ae16b2bdf.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>7 . Compute the Mean-squared Error</h2>

1\.  Compute and print the mean-squared error of the predictions:
```
ratesAndPreds = ratings.map(lambda r: ((r[0], r[1]), r[2])).join(predictions) 
MSE = ratesAndPreds.map(lambda r: (r[1][0] - r[1][1])**2).reduce(lambda x, y: x + y)/ratesAndPreds.count()
print("Mean Squared Error = " + str(MSE))
```

2\.  Run the cell. The output should look like:

![using-spark-mlib-2](https://user-images.githubusercontent.com/21102559/40943056-e6ffa428-681d-11e8-9e71-c9b7b22bed6b.png)


### Result

You have used Spark MLlib to execute a recommender engine using alternating least squares (implemented by the ALS class in Spark). You also computed the mean-squared error of the model.

You are finished!
