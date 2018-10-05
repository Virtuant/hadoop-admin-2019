## Computing a K-Nearest Neighbor

### About This Lab

**Objective:** To use NumPy, pandas, and matplotlib modules to compute the K-Nearest neighbor of a point on a graph

**Successful outcome:** A Python function that computes the K-nearest neighbor of a point from four sets of randomly generated points, along with a plot showing the data

**Lab Files:** `~/labs`

---
Steps
---------

<!-- STEP -->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. Open the K-Nearest-Neighbor Notebook</h2>

1\.  Open your Web browser to the IPython Notebook/Jupyter `Dashboard` page.

2\.  Navigate into the `labs` folder.

3\.  You should see a notebook named `K-Nearest Neighbor`. Click on it to open the notebook.


<!-- STEP -->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Import the Modules</h2>

1\.  Notice the notebook has four cells, and the first cell is empty.

2\.  Within the first cell, import the `random` module.

3\.  Import the `operator` module.

4\.  Import the `numpy` module as `np`.

5\.  Import the `pandas` module as `pd`.

6\.  Import `matplotlib.pyplot` as `plt`.

7\.  The code uses `DataFrame` from `pandas`, so add the following:
```
from pandas import DataFrame
```

8\.  The first cell should now look like the following:

![computing-k-nearest-neighbor-1](https://user-images.githubusercontent.com/21102559/40942604-ba569eaa-681c-11e8-9d54-fa3bf015bf14.png)

9\.  Run the cell to verify your imports are correct, and fix any errors if necessary.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Compute the Distance Between Points</h2>

1\.  Notice in the second cell of the notebook that a function has been partially written named `knn`.

2\.  After the "#distance calculation" comment, add the following code that computes the distances between the test point and each point in the data set:

```
diffMat = tile(X, (dataSetSize,1)) - dataSet 
sqDiffMat = diffMat**2
sqDistances = sqDiffMat.sum(axis=1) 
distances = sqDistances**0.5
```

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Sort the Distances</h2>

1\.  After the "#sort results" comment, sort the `DataFrame` named `distances` using the `sort` function:
```
distances.sort()
```

2\.  Your second cell should look like this:

![computing-k-nearest-neighbor-2](https://user-images.githubusercontent.com/21102559/40942605-ba65bcbe-681c-11e8-92d7-5fee084862b9.png)

3\.  Run the code in your second cell. There will be no output, but you need to run the code so that the function definition is processed by IPython/Jupyter.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2> 5. Generate the Data</h2>

1\.  Notice in the third cell of the notebook that 100 random `(x,y)` points are generated, with `x` and `y` between `0` and `100`.

2\.  After the "Define a test point" comment, add the following line of code that creates a DataFrame to represent the point `(15, 30)`:

```
test_point = DataFrame({"x": [15], "y": [30]})
```

3\.  The third cell should now look like:

![computing-k-nearest-neighbor-3](https://user-images.githubusercontent.com/21102559/41186726-64e56780-6b69-11e8-86d6-5a6001f95b43.png)

4\.  Run the code in the third cell so it is processed by IPython/Jupyter


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>6. Invoke the `knn` Function</h2>

1\.  In the fourth cell after the "#Invoke the knn function" comment, set `k` equal to 5:
```
k = 5
```

2\.  Invoke the `knn` function with the test point, the random data, and the value of k: 
```
result_label = knn(test_point, data, k)
```

3\.  Print the result:
```
print "***Result = %s***" % result_label
```

4\.  The fourth cell should look like:

![computing-k-nearest-neighbor-4](https://user-images.githubusercontent.com/21102559/41186727-6508ea34-6b69-11e8-8e99-557b1c75ac3e.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>7. Run the Cell</h2>

1\.  The code is now ready to run. Run the code in the fourth cell. The result will display in a `textarea` below the cell, and the output is random:
```
Result = A
```

2\.  Run the code in the third cell again to generate new random data, then run cell four again to see the new results. Do this a few times and you should get varying results.


<!-- STEP -->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>8. Plot the Results</h2>

1\.  Notice the fifth cell is written for you and contains code to plot the 100 random points, along with the test point. Run the fifth cell.

The output should look similar to the following (the yellow point is the test point):

![lab-computing-k-nearest-neighbor-5](https://user-images.githubusercontent.com/21102559/41167765-e66ab04c-6b11-11e8-980e-23a4f6345b62.png)

2\.  Run the code on the entire page again by selecting `Cell -\> Run All` from the menu. The random points change, as should the result of the nearest neighbor and the corresponding plot.


<!-- STEP -->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>9. Change the Value of K</h2>

1\.  Notice the value of `k` is defined in the fourth cell. This allows you to change the value of k while still using the same random points. Change the value of `k` from `5` to `15`, then run the code only in cell four. Does the result change?

2\.  Try other values of `k` on the same data to see the effect of the value of `k`. 


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>10. Change the Test Point</h2>

1\.  Change the test point back up in the third cell and run the code again.

2\.  Feel free to experiment by changing the test point, the range of the random points, and the value of `k`.

### Result

In this lab, you computed the nearest neighbor of a single point based on four different groups of randomly- generated points.

You are finished!
