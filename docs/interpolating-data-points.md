## Interpolating Data Points

### About This Lab

**Objective:** To use SciPy to interpolate a set of fixed data points

**Successful outcome:** You will have plotted a cubic spline interpolation of the function `cos(x^2)`

**File location:** `~/labs`

---
Lab Steps
---------



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. Create a Notebook</h2>

1\.  Make sure you are still in the `labs` folder.

2\.  Click the "New Notebook" button to create a new Jupyter Python 2 notebook.

3\.  Change the title to "SplineInterpolation". 



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Import the Modules</h2>


1\.  In the first cell, import the following modules and functions:
```
import numpy as np
|import matplotlib.pyplot as plt
from scipy.interpolate import interp1d
```

> Note  The name of the object is interp1d with the number "1", not the letter "l".

2\.  Run the cell.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Define the Fixed Data Points</h2>

1\.  Use the `np.linspace` function to generate 10 evenly-spaced values for `x`:
```
x = np.linspace(0,10,10)
```

2\.  View the values `x` by displaying `x` and running the cell. The output should look like:

![interpolating-data-points-1](https://user-images.githubusercontent.com/21102559/40942682-fd80776e-681c-11e8-8780-24b4525e0155.png)

3\.  Set `y` equal the function `cos(x**2)`:
```
y = np.cos(x**2)
```

4\.  View the values of `y` by displaying `y` and running the cell. The output should look like:

![interpolating-data-points-2](https://user-images.githubusercontent.com/21102559/40942683-fd91ae12-681c-11e8-80ba-29c08a8448e5.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Plot the Data Points</h2>

1\.  Let's look at the data points by plotting them in a new cell in your notebook: 

```
plt.plot(x,y,'o')
``` 

Your plot should look like:

![interpolating-data-points-3](https://user-images.githubusercontent.com/21102559/40942684-fd9f580a-681c-11e8-97f4-079d8662e875.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>5. Interpolate the Points</h2>

1\.  In a new cell, define a function `f` that is a cubic spline interpolation of the data points:
```
f = interp1d(x, y, kind='cubic')
```

2\.  Define a new set of `x` values for `f`:
```
x_values = np.linspace(0, 10, 40)
```

3\.  Plot **f**:
```
plot(x_values,f(x_values),'-')
```

4\.  Run the cell

5\.  `f` should look like:

![interpolating-data-points-4](https://user-images.githubusercontent.com/21102559/40942685-fdac3f20-681c-11e8-9f2b-be1c53aa371f.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>6. Plot the Two Results Together</h2>

1\.  In a new cell, plot the data points and the interpolated function on the same plot:
```
plt.plot(x,y,'o',x_values,f(x_values),'-') 
plt.legend(['data points', 'cubic'], loc='best') 
plt.show()
```

The plot should look like:

![interpolating-data-points-5](https://user-images.githubusercontent.com/21102559/40942686-fdbae8fe-681c-11e8-92cc-a02575e00596.png)

### Result

You have used the interp1d class in scipy.interpolate to create a function based on a fixed set of data points.

You are finished! 
