## Pandas Library

**Objective:** Understand basic fundamentals of working with the Pandas Library

**Successful outcome:** Complete the following steps and ask questions when needed

**File location:** `~/labs`

---
### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1. Open the pandas Notebook</h2>

1\.  Create a new Notebook in IPython.

2\.  Change the name of the Notebook to `pandasDemo`.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Import the Modules</h2>

1\.  In the first cell, import the `pandas` and `numpy` modules:
```
import pandas as pd 
import numpy as np 
```

2\.  Import a couple of commonly used objects:
```
from pandas import DataFrame, Series
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Working with Series</h2>

1\.  A Series is a one-dimensional array-like object containing an array of data of any NumPy data type. For example:
```
x1 = Series([12,-5,7,15,-2]) 
x1 
```

2\.  Run the cell. Confirm that each entry in the Series has an index, and the data type of the objects in `x1` is `int64`.

3\.  You can print just the values:
```
x1.values
```

4\.  You can print just the indices:
```
x1.index
```

5\.  You can specify a value by its index:
```
x1[2]
```

6\.  You can specify multiple indexes and in any order you want using
    double square brackets:
```
x1[[2,0,3]]
```

7\.  You can specify a Boolean expression within the square brackets:
```
x1[x1 < 0]
```

8\.  You can perform mathematical operations on each value in a Series. Try the following:
```
print x1 * 2 
print x1**2
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Creating a Series from a `dict`</h2>

1\.  You can create a Series from a dict:
```
sdata = {'Ohio': 35000, 'Texas': 71000, 'Oregon': 16000, 'Utah': 5000} 
x2 = Series(sdata)
x2 
```

2\.  You can create a Series from a dict and specify the indexes you want:

> Note  California is `NaN` and Utah does not appear in `x3`. Also notice the data type of the numeric column changed from `int64` to `float64` because of the introduction of a `NaN` value.

```
<blockquote>
**Reference**<br>
You can view the details of why at:<br>
<a>http://pandas.pydata.org/pandas-docs/stable/gotchas.html#nan-integer-na-values-and-na-type-promotions</a>
</blockquote>
```

```
states = ['California', 'Ohio', 'Oregon', 'Texas'] 
x3 = Series(sdata, index=states)
x3 
```

3\.  Use the `isnull` function on the `x3` series:
```
pd.isnull(x3)
```

4\.  You can drop nulls from a Series using the dropna function:
```
x3.dropna()
```

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>5. Plotting Series</h2>

1\.  The Series class has a `plot()` function:
```
x2.plot?
```

2\.  Plot the `x2` Series:
```
x2.plot(kind='bar')
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>6. Constructing DataFrames</h2>

1\.  A DataFrame represents a tabular, spreadsheet-like data structure containing an ordered collection of columns. You can create one from `dicts` of equal length:
```
data = {'state' : ['California', 'Texas', 'Oregon', 'Ohio'], 
        'population' : [38.3,26.4,3.9,11.5]}
f1 = DataFrame(data) 
f1
```

2\.  Notice in the output of `f1` that the population was chosen as the first column. You can specify the order of columns using the `columns` attribute:
```
f2 = DataFrame(data,columns=['state','population']) 
f2
```

3\.  If a column listed in `columns` does not exist in the data, then the values will be `NaN`:
```
f3 = DataFrame(data,columns=['state','population','percent']) 
f3
```

4\.  Adding a `percent dict` resolves that issue:
```
data = {'state' : ['California', 'Texas', 'Oregon', 'Ohio'], 
        'population' : [38.3,26.4,3.9,11.5],
        'percent' : [11.9,8.0,1.2,3.7]}
f4 = DataFrame(data,columns=['state','population','percent']) 
f4
```

5\.  You can specify a custom index as well using the `index` attribute:
```
f5 = DataFrame(data,columns=['state','population','percent'], 
                index=[31,28,33,17])
f5
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>7. Accessing Cells</h2>

1\. The DataFrame object has a columns function for retrieving the column names:
```
f5.columns
```

2\. You can retrieve a single column using either the column name (like in a dict):
```
f5['population']
```

Or you can reference a column like its an attribute:
```
f5.population
```

3\. You can perform operations and create Boolean expressions:
```
f5.population > 20
```

4\. Use the ix attribute to access a specific cell by row and column:
```
f5.ix[28,'state']
```

5\. You can slice a range of rows:
```
f5[:2]
f5[1:]
```

f.  You can transpose a DataFrame:
```
f5.T
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>8. Sorting DataFrames</h2>

1\.  Use `sort_index` to sort the rows of a DataFrame by the index:
```
f5.sort_index()
```

2\.  You can also specify which column (or columns) to sort by:
```
f5.sort_index(by='state')
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>9. Statistical Functions</h2>

1\.  You can perform operations on a DataFrame. To add up the values in every column:
```
f5.sum()
```

2\.  The `describe()` function shows statistics about the DataFrame:
```
f5.describe()
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>10. Plotting DataFrames</h2>

1\.  DataFrame has a `plot()` function:
```
f5.plot?
```

2\.  For example:
```
f5.plot(kind='bar')
```

### Result

You are finished!
