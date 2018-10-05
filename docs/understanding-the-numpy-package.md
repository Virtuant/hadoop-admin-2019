## Understanding the NumPy Package

**Objective:** Learning more about NumPy and its fundamental functionality of this Python based tool.

**Successful outcome:** Follow along with the steps below and be sure to ask question if needed. And hae fun!

**File location:** `~/labs`

---
### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1. Create a Notebook</h2>

1\.  Create a new Notebook in IPython in the `labs` folder.

2\.  Change the name of the Notebook to NumPy Demo.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Import the Module</h2>

1\.  In the first cell, import the `numpy` module:

```
import numpy as np
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. Define an ndarray</h2>

1\.  An `ndarray` can be created various ways. The following `ndarray` is created using the `numpy.array` function and passing in a list:

```
a = np.array([4,5,6,7,8,9]) 
print a
print type(a)
print a.dtype
print a.shape
```

2\. Run the cell. Notice the data type of `a`, and also the data type of the values in `a`.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>4. The arange Function</h2>

1\.  A simple way to define an `ndarray` filled with integers is `arange`:

```
b = arange(6) 
print b
print type(b)
```

2\.  Notice `b` is an `ndarray` with integers `0 -- 5`.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>5. Array Operations</h2>

1\.  If arrays are the same size, you can perform scalar operations on them. For example:

```
a * b
```

2\.  You can perform an operation on each element in an `ndarray`:

```
a ** 2
```



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Indexing and Slicing</h2>

1\.  Indices work as expected with an `ndarray`. They are 0-based:

```
print a[1]
```

2\.  The slice notation also works on an ndarray object:

```
print a[2:4]
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. Multi-dimensional Arrays</h2>

1\.  The "nd" in `ndarray` stands for "n-dimensional", so it should be no surprise that you can have multi-dimensional `ndarrays` of any dimension:

```
m = arange(1,13).reshape(3,4) 
print m
print m.shape 
```

2\.  An index for `m` requires two values since it is a 2-dimensional `ndarray`:

```
m[1,2]
```

3\.  Slicing can be done at each dimension of the `ndarray`. For example, the following slice is the first two rows:

```
m[:2]
```

4\.  In a 2-dimensional array, you can slice rows and columns:

```
m[:2,:3]
```

5\.  Boolean expressions can be applied to the values in the index of an `ndarray`. For example:

```
m[m > 5] = -1 
m
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>8. Fancy Indexing</h2>

1\.  NumPy has a concept known as fancy indexing for selecting specific rows from an `ndarray`. Let's start with an array filled with integers:
```
m2 = arange(60).reshape(6,10) 
m2 
```

2\.  The following "fancy" index syntax selects the third, fourth and first row from `m2`, in that order:

```
m2[[2,3,0]]
```

3\.  You can use negative values to select backwards from the end of the array, where -1 is the last row, and so on:

```
m2[[-1,-2, 2]]
```

4\.  Notice you can easily transpose an ndarray object:

```
m2.T
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>9. Linear Algebra</h2>

1\.  Let's perform a dot product of two arrays. Start by defining the following `(10,4)` array of integers:

```
m3 = arange(40).reshape(10,4) 
m3 
```

2\.  Now take the dot product of `m2` (which is size `6x10`) and `m3` (which is size `10x4`): 

Notice, as expected, the product is a `6x4` array.

```
product = np.dot(m2,m3) 
product
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>10. Plotting Image Arrays</h2>

1\.  The matplotlib library has a function named `imshow()` that prints an `ndarray` as an image (where each value in the array represents an RGB color). Let's display the product `ndarray` as an image:

```
import matplotlib.pyplot as plt 
plt.imshow(product) 
```

You should see a colorful rainbow-like image.

![understanding-the-numpy-package-1](https://user-images.githubusercontent.com/21102559/40943020-cba54b38-681d-11e8-851f-f14b46dd37bc.png)

2\.  Load an image and view its `ndarray` object. Start by adding the following import:

```
import matplotlib.image as mpimg
```

3\.  There is a PNG file named `hortonworks.png` that is in `notebooks/demos/`
You should see a large array of floats.

```
hwx = mpimg.imread('hortonworks.png') 
hwx 
```

4\.  Plot the image:

```
plt.imshow(hwx)
```

You should see the PNG file that was loaded:

![understanding-the-numpy-package-2](https://user-images.githubusercontent.com/21102559/40943021-cbb47c02-681d-11e8-9c0f-5633f483e355.png)

### Result

You are finished!
