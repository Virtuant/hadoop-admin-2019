## Creating a Spam Classifier using Spark MLlib

## About this Lab


**Objective:** Become familiar with using Spark MLlib to run data science algorithms on a Hadoop cluster.

**Successful Outcome:** You will have created a spam classifier with MLlib.

**Lab:** `~/labs`


---
Steps
---------

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1. Create a New Notebook</h2>

1\.  From your web browser, go to the IPython Notebook/Jupyter page.


If necessary, run the spark start script:

<!-- ORGINAL
/root/ds/scripts/start_pyspark.sh
-->

```
/[user-name]/labs/start_pyspark.sh
```

2\.  Click on the `labs` folder of IPython.

3\.  Click the **New Notebook** button to create a new Python 2 Notebook.

4\.  Change the name of the Notebook to `SpamClassifier`:

![creating-a-spam-classifier-using-spark-mllib-1](https://user-images.githubusercontent.com/21102559/40942618-cadd1a92-681c-11e8-9f52-3c5028e8f662.png)

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Load the labeled data set to train the classifier</h2>

1\. The labeled data is in your VM at:
```
ls /[user-name]/labs/spamClassData
```

The files themselves are not labeled as either spam or not, but there is a separate file, `SPAMTrain.txt`, that has the label `1` for spam, `0` for not-spam for each filename.
It is in the directory "label" at the same path.

On your Notebook, create empty lists to hold the `spam/non-spam` file names.

```
spamFiles = [] 
notSpamFiles = []
```

2\.  Read the `SPAMTrain.txt` file to find out if the file in question is labeled spam or not: spam is labeled 0 , non-spam is labeled 1


Append this code to the cell creating the `spamFiles` and `notSpamFiles` lists, then run the cell

```
spamFiles=[] 
notSpamFiles=[]
f=open('/[user-name]/labs/spamClassData/label/SPAMTrain.txt', 'r') 
   for line in f:
	  if int(line[0]) == 1: 
		r = line[2:]

notSpamFiles.append('/[user-name]/labs/spamClassData/'+r.rstrip('\n')) 
   elif int(line[0]) == 0:
		r = line[2:]

spamFiles.append('/[user-name]/labs/spamClassData/'+r.rstrip('\n')) 
print len(notSpamFiles)
print len(spamFiles)
```

![creating-a-spam-classifier-using-spark-mllib-2](https://user-images.githubusercontent.com/21102559/40942619-caecad18-681c-11e8-90f2-639877d5d170.png)

3\.  Create a list with the contents of the files in `spamFiles`

```
spams = []
for file in spamFiles: 
	f = open(file,"r")
	spams.append(f.read()) 
print len(spams)
```

4\.  In the same cell, create a list with the contents of the files in `nonSpamFiles`, then run the cell

```
notSpams = []
for file in notSpamFiles: 
	g = open(file,"r")
	notSpams.append(g.read()) 
print len(notSpams)
```

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. Import MLlib modules to do Logistic Regression as our classifier.</h2>

Logistic regression requires as input a labeled training set in the form of a vector of doubles.

We will also make use of the regression library's LabeledPoint to prepare our data in this format.

```
from pyspark import SparkContext, SparkConf
conf = SparkConf().set('spark.executor.instances', 1). \ 
	set("spark.executor.memory", "4g")
sc = SparkContext("yarn-client", conf=conf)
from pyspark.mllib.regression 
import LabeledPoint from pyspark.mllib.feature 
import HashingTF from pyspark.mllib.classification 
import LogisticRegressionWithSGD
```

1\.  Run the cell.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>4. Create training data RDDs</h2>

1\.  Create a spam emails object

```
spam = sc.parallelize(spams)
```

2\.  And create a non-spam/normal emails object

```
normal = sc.parallelize(notSpams)
```

5\. Create a `HashingTF` Instance

We will use this instance to map the email text to uniformly length vectors all with 1,000,000 features (i.e. S =1000000)

```
tf = HashingTF(numFeatures = 1000000)
```

Each email is split into words, and each word is mapped to one feature, either spam or normal.

```
spamFeatures = spam.map(lambda email: tf.transform(email.split(" "))) 
normalFeatures = normal.map(lambda email: tf.transform(email.split(" ")))
```

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Create `LabeledPoint` Datasets</h2>

Use positive (spam) and negative (normal) examples.

1\.  The positive and negative `LabeledPoint` objects are key/value

```
positiveExamples = spamFeatures.map(lambda features: LabeledPoint(1, features))
negativeExamples = normalFeatures.map(lambda features: LabeledPoint(0, features))
```

2\.  Union the datasets together into our labeled training data

```
trainingData = positiveExamples.union(negativeExamples) 
trainingData.cache()
```

The last lined is cached since Logistic Regression is an iterative algorithm.
```
trainingData.take(1)
```

Your result should look something like:

![creating-a-spam-classifier-using-spark-mllib-3](https://user-images.githubusercontent.com/21102559/40942620-cafaeed2-681c-11e8-8db2-655d870c9481.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. Run Logistic Regression using the SGD algorithm</h2>

1\.  Create a model trained on the prepared dataset.
```
model = LogisticRegressionWithSGD.train(trainingData)
```

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>8. Test on a positive example (spam) and a negative one (normal).</h2>

1\. First apply the same `HashingTF` feature transformation to get vectors (i.e. `tf.transform`), then apply the model (i.e. `model.predict`).

```
posTest = tf.transform("Poker for money against real playersGet your favorite Poker action!".split(" "))
negTest = tf.transform("Hi Dad, I started studying Spark the other...".split(" "))
print "Prediction for positive test example: %g" % model.predict(posTest) 
print "Prediction for negative test example: %g" % model.predict(negTest)
```

![creating-a-spam-classifier-using-spark-mllib-4](https://user-images.githubusercontent.com/21102559/40942621-cb0e20ec-681c-11e8-8131-1f38eacf74a6.png)


### Result

You have now created a spam classifier using MLlib.

You are finished!
