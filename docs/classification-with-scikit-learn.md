## Classification with Scikit-Learn

### In this Lab

**Objective:** To understand how SciKit forms its opinions

**During this Lab:** Perform the following steps

**Lab Files:** `~/labs`

----

### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. Open the Notebook</h2>

1\.  In a folder of IPython Notebook, open the notebook named *Scikit-Classification_Demo*.

2\.  The steps for this lab are in the Notebook. Run each cell one at a time and discuss the results along the way. Or you can work through the steps below manually.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Use Support Vector Classification</h2>

1\.  First we need to fit some training data. We will use some following data points:

```
training_data = 
[[1,1],[2,2],[1,3],[0.5,0],[0.3,3],[0.9,0.8],[0.9,1.2],[1.1,0.8],[1.8,1.5],[0 .8,2.1],[3.1,3.3],[3.2,2.9],[3,5],[2.9,4.5],[0.3,0.4],[3,3],[4,4],[3.5,3.1],[ 3.9,4.2],[2.5,2.9]]
labels = 
['a','a','a','a','a','a','a','a','a','a','b','b','b','b','b','b','b','b','b', 'b']
```

2\.  Run the first three cells to generate the plot of the training data:

![classification-with-scikit-learn-1](https://user-images.githubusercontent.com/21102559/40942057-e7f6fbcc-681a-11e8-8ad2-8ee2ce21852c.png)

3\.  In the cell labeled "Step 2: Instantiate the classifier here", import the svm class:
```
from sklearn import svm
```

4\.  Instantiate an **SVC** (support vector classification) object with the **rbf** kernel:

```
svc = svm.SVC(kernel='rbf')
```

5\.  Fit the training data using the **SVC** object:
```
svc.fit(training_data, labels)
```

6\.  Run the cell. The output should look like:
```
SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0, degree=3, gamma=0.0, kernel='rbf', max_iter=-1, probability=False, random_state=None, shrinking=True, tol=0.001, verbose=False)
```

7\.  Skip cells 5 and 6 for now.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Predict New Data Points</h2>

1\.  Let's see where a new data point will be classified:

```
test1 = [2.3, 2.5]; 
svc.predict([test1])
```

This point is right between the two groups. Notice it ends up being classified as `a`:

```
array(['a'],
          dtype='|S1')
```

2\.  Try the point `[2.4, 2.5]` and see how it is categorized.

3\.  Run the cell that plots the test points with the training data. It should look like the following:

![classification-with-scikit-learn-2](https://user-images.githubusercontent.com/21102559/40942062-ea7a7c48-681a-11e8-8341-db32c2efd574.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Gaussian Naive Bayes</h2>

1\.  Let's fit some data using a different machine learning algorithm: Gaussian Naive Bayes. The `sklearn.naive_bayes` module contains a GaussianNB class:
```
from sklearn.naive_bayes import GaussianNB 
gnb = GaussianNB() 
```

2\.  Run the cell.

3\.  The next cell generates 55 random points and labels them randomly as "hot", "warm", "cool" or "cold":

```
from pandas import DataFrame
#Generate random points. N should be even
N = 55
#labels = ["hot", "warm", "cool", "cold"] * 25
label_cold = ["cold"]*(N/2)
label_hot = ["hot"] *(N/2)
data_cold = DataFrame(np.random.randint(0, 55, size=((N/2), 2)), columns = ["x", "y"], index = label_cold)
data_hot = DataFrame(np.random.randint(45, 100, size=((N/2), 2)), columns = ["x", "y"], index = label_hot)
data = data_cold.append(data_hot) 
data
```

4\.  Run the cell. Notice the `data.count` output shows the points that were generated and their label.

5\.  Run the cell that shows the plot:

The output will vary, but notice the "hot" points are red, "warm" points are yellow, "cool" points are green, and "cold" points are blue:

```
labels = label_cold+label_hot 
colors = {'hot': 'r', 'cold': 'b'}

ax = data_cold.plot(x='x', y='y', kind='scatter', color='b') 
data_hot.plot(x='x', y='y', kind='scatter', color='r', ax=ax)
```

![classification-with-scikit-learn-3](https://user-images.githubusercontent.com/21102559/40942065-ec569178-681a-11e8-8c23-3b9cc1b7934f.png)

6\.  In the empty cell below the plot, fit the training data using the GNB algorithm:
```
gnb.fit(data,labels)
```

7\.  Run the cell. The output will simply be:
```
GaussianNB()
```

8\.  Predict the classification of a point:

> Note: Since the points are random, the result will vary each time you regenerate the training data and refit it. Keep trying other points. Try to predict the outcome of a point based on your training data, and then test your guess.

```
gnb.predict([[15,30]])
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>5. Compare SVM to GNB</h2>

1\.  Fit the random data points using the SVC algorithm:
```
svc.fit(data,labels)
```

2\.  Predict the same test point and see if you get the same answer as GNB:
```
svc.predict([[15,30]])
```

3\.  Try comparing different points and see if you can get different classifications with the same test point.

### Result

You are finished!
