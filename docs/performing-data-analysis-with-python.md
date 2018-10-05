## Performing Data Analysis with Python

### About This Lab

**Objective:** To use various Python librabries to analyze data

**Successful outcome:** You will have analyzed the New York Stock Exchange data using DataFrames and the various functions in pandas and NumPy

**File locations:** `~/labs`

---
### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. View the Data Files</h2>

1\.  View the contents of the `stock_prices` folder:

```
ls -la
```

Output:

```
total 116044
-rw-r--r-- 1 root root 40990992 NYSE_daily_prices_A.csv.gz 
-rw-r--r-- 1 root root 32034760 NYSE_daily_prices_B.csv.gz 
-rw-r--r-- 1 root root 45790256 NYSE_daily_prices_C.csv.gz
```

3\. Notice the files are compressed. The text below shows what they look like: there is a header row, and then the stock price history in a comma-separated format:

```
exchange,stock_symbol,date,stock_price_open,stock_price_high,stock_price_low, stock_price_close,stock_volume,stock_price_adj_close 
NYSE,AEA,2010-02-08,4.42,4.42,4.21,4.24,205500,4.24 
NYSE,AEA,2010-02-05,4.42,4.54,4.22,4.41,194300,4.41 
NYSE,AEA,2010-02-04,4.55,4.69,4.39,4.42,233800,4.42
```

You are going to load these three compressed files into a pandas DataFrame.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Create a Notebook</h2>

1\.  In your browser, go to the `Dashboard` page of IPython Notebook/Jupyter.

2\.  Navigate to the `labs` folder.

3\.  Click the "New Notebook" button to create a new Jupyter Python 2 notebook.

4\.  Change the title to "StockPrices". 


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Import the Libraries</h2>


1\.  In the first cell, add the following imports:
```
import pandas as pd
import numpy as np
from pandas import DataFrame, Series 
import glob
```

2\.  Run the first cell so the imports are processed by IPython.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Read in the Data Files</h2>

1\.  In the second cell, create a variable named `stock_prices` that contains all the CSV files in the `stock_prices` folder:
```
stock_prices = glob.glob('stock_prices/*.csv.gz') 
stock_prices 
```

2\.  Run the second cell. The output should be an array of the three CSV filenames in the `stock_prices` folder:

![performing-data-analysis-with-python-1](https://user-images.githubusercontent.com/21102559/40942820-4feef368-681d-11e8-9840-9138fa542e72.png)

---
> Note  If you do not see these files, perhaps you did not create the notebook in the labs folder. Notice the code above uses a relative path.<br> 
To determine which folder the notebook is in, enter the !pwd command in a cell and run it:

![performing-data-analysis-with-python-2](https://user-images.githubusercontent.com/21102559/40942822-501d3a52-681d-11e8-9e18-82a01b8a0e58.png)

If your present working directory is different, then modify the path to `stock_prices` accordingly.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>5. Load the Data into a DataFrame</h2>

1\.  In a new cell, create a new `DataFrame` to contain the stock price data:

```
stock_data = DataFrame()
```

2\.  Add a `for` loop that iterates through the CSV files:

```
for myFile in stock_prices :
```

3\.  Using the `read_csv` function of pandas, read in the current file using the following arguments:

```
current_data = pd.read_csv(myFile,compression='gzip')
```

4\.  Within the for loop, append the `current_data` DataFrame to the `stock_data` DataFrame:

```
stock_data = stock_data.append(current_data)
```

5\.  Outside the for loop, display the `stock_data`:

```
stock_data
```

6\.  Your cell should look like:

![performing-data-analysis-with-python-3](https://user-images.githubusercontent.com/21102559/40942823-5042957c-681d-11e8-8b44-51df19b80eac.png)

7\.  Run the cell. This may take a minute to process the three CSV files. 
The output should be a table that looks similar to the following:

![performing-data-analysis-with-python-4](https://user-images.githubusercontent.com/21102559/40942824-505d4f5c-681d-11e8-8aa9-7a2f4f3d08b1.png)

8\.  Scroll down to the bottom of this table. Notice the output displays the total number of rows and columns. Verify your DataFrame has 2,138,044 rows and 9 columns.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>6. Selecting Rows and Columns</h2>

1\.  A *DataFrame* is very similar to a database table, and in many scenarios will access the data in a DataFrame as you would using SQL to access a database table.

For example, if you wanted data in a single column, you would enter:

```
stock_data['stock_price_high']
```

2\.  Run the cell. You will see just the one column (returned as a **Series** object), along with the index:

![performing-data-analysis-with-python-5](https://user-images.githubusercontent.com/21102559/40942825-509ccfa6-681d-11e8-8329-717e083442c4.png)

3\.  To view multiple columns, enter the column names into an array. To demonstrate, run the following command in a new cell:

```
stock_data[['stock_symbol','date','stock_price_high']]
```

4\.  This time you should see the output of a **DataFrame** (in a nicer format than the single column which was only a **Series**):

![performing-data-analysis-with-python-6](https://user-images.githubusercontent.com/21102559/40942826-50b4261a-681d-11e8-8336-4485b4fb64ba.png)

5\.  You can also specify a Boolean expression, which is similar to a `WHERE` clause in SQL. Run the following command in a new cell, which displays stocks that had a daily volume of more than ten million:
```
stock_data[stock_data['stock_volume'] >= 10000000]
```

6\. Run the previous command again, but this time use the `tail()` function to limit the results to 20:
```
stock_data[stock_data['stock_volume'] >= 10000000].tail(20)
```



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>7. Analyzing Stock Data by Symbol</h2>

1\.  Just like `GROUP` BY in SQL, the DataFrame object has a groupby function for grouping results.

To demonstrate, run the following command in a new cell:
```
stock_data.groupby('stock_symbol').size()
```

2\.  The output should look like:

![performing-data-analysis-with-python-7](https://user-images.githubusercontent.com/21102559/40942827-50e0e25e-681d-11e8-9fca-3e5d26bf157b.png)

> Note  The numbers next to each stock symbol are historically the number of days that the stock was traded (at least in the CSV files).

3\.  Let's use the groupby function to compute statistics on individual stocks.
For example, run the following command in a new cell, which displays the maximum high price of a stock:
```
stock_data.groupby('stock_symbol')['stock_price_high'].max()
```

4\.  The output should look like:

![performing-data-analysis-with-python-8](https://user-images.githubusercontent.com/21102559/40942828-510123f2-681d-11e8-9915-8fa7115f2640.png)

5\.  Run a similar command that displays the minimum value of the `stock_price_low column` for each `stock_symbol`.

6\.  Run a command that displays the average `stock_volume` of each stock.

> HINT: Use the `mean()` function.

7\. You can display multiple statistical values in a `DataFrame` using NumPy and the `agg` function.
Try running the following command:

```
stock_data.groupby('stock_symbol').agg({'stock_price_high':[np.max, np.min, np.mean]})
```

The output should look like:

![performing-data-analysis-with-python-9](https://user-images.githubusercontent.com/21102559/40942855-5b158eaa-681d-11e8-9d0b-ee894a2e3ec1.png)




<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>8. Working with a Specific Stock</h2>

1\.  Run the following command, which displays the maximum closing price of just the `AYI` stock:

```
stock_data[(stock_data['stock_symbol'] == 'AYI')].stock_price_close.max()
```

The result should be `$65.03`. 




<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>9. Plotting DataFrames</h2>

1\.  The `DataFrame` object has a `plot` function, but plotting an entire `DataFrame` is likely to crash your machine.

However, you can specify a single stock and plot information about its history. Try the following commands, which are similar to your previous one above:

```
stock_data['date']=pd.to_datetime(stock_data['date']) stock_data[(stock_data['stock_symbol'] == 'AYI')].plot(x="date", y="stock_price_close")
```

2\.  The plot should look like:

![performing-data-analysis-with-python-10](https://user-images.githubusercontent.com/21102559/40942856-5b2d0882-681d-11e8-9424-df90751fef8c.png)

3\.  Try plotting the `stock_volume` column of the `ABA` stock. 




<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>10. Defining a MultiIndex</h2>

1\. There are several ways to create a `MultiIndex` that contains both the stock symbol and date, but the simplest technique is to create it as the data is being read in.

In a new cell at the bottom of your `StockPrices` notebook, define a new `DataFrame`  named `stock_data2`:

```
stock_data2 = DataFrame()
```

2\.  Define a `for` loop that iterates through the three CSV files:

```
for myFile in stock_prices :
```

3\.  Within the `for` loop, use the pandas `read_csv` function like you did before, except add an `index_col` argument that consists of the multiple columns that are to be a part of the index.

In our data, that is the 2nd and 3rd columns:

```
current_data = pd.read_csv(file,index_col=[1,2],compression='gzip')
```

4\.  Still in the `for` loop, add an **if** statement that assigns `current_data` to `stock_data2`:

```
if stock_data2.empty : 
    stock_data2 = current_data
```

5\.  In the `else` block, append the `current_data` DataFrame to the `stock_data2` DataFrame:

```
else :
    stock_data2 = stock_data2.append(current_data)
```

6\.  Display `stock_data2` outside the for loop. Your cell should look like:

![performing-data-analysis-with-python-11](https://user-images.githubusercontent.com/21102559/40942857-5b3e94f8-681d-11e8-9fb1-657e803adc0f.png)

7\.  Run the cell. The output should look like:

![performing-data-analysis-with-python-12](https://user-images.githubusercontent.com/21102559/40942858-5b4d943a-681d-11e8-8963-fd6be89dd764.png)

> Note  the first two columns are `stock_symbol` and `date`, and that the prices of each unique `stock_symbol` are now grouped together.




<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>11. Analyze the Stock Prices</h2>

1\.  Let's perform some more analysis. Try running the following command:

```
stock_data2.stock_price_high.max()
```

> Note  the output is a single row, and is probably not that interesting of a value statistically because it is a global result and our data is naturally grouped by stock symbol.

2\.  Run the same command again in a new cell, except this time specify the `level` of the computation as 0:

```
stock_data2.stock_price_high.max(level=0) 
```

> Notice this time the result is per unique `stock_symbol`, which is a much more interesting result:
![performing-data-analysis-with-python-13](https://user-images.githubusercontent.com/21102559/40942860-5b60c4b0-681d-11e8-8919-318f9df5fb8e.png)

3\.  Similarly, write a command that displays the minimum of the `stock_price_low column` for each stock symbol. The output should look like:

![performing-data-analysis-with-python-14](https://user-images.githubusercontent.com/21102559/40942861-5b728b3c-681d-11e8-8f45-b5be901a30ff.png)

4\.  See if you can write a command that sums up the `stock_volume` column for each stock symbol. The output should look like:

![performing-data-analysis-with-python-15](https://user-images.githubusercontent.com/21102559/40942862-5b863cea-681d-11e8-98d0-b12aa85a6910.png)

5\.  Now write a command that displays the total sum of all `stock_volume` columns in the `DataFrame`.
    
The result is 1,821,701,605,000.




<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>12. Accessing Specific Indexes</h2>

1\.  You can use the ix function to view the rows for a specific stock symbol (or collection of stock symbols).

For example, run the following command in a new cell to view just the 'CLI' stock prices:

```
stock_data2.ix['CLI']
```

2\. This syntax allows you to find results for a specific stock. For example, the maximum `stock_price_high` for `CLI` is found by:
```
stock_data2.ix['CLI'].stock_price_high.max() 
```

Run the command. You should get `$56.52`.

3\.  Plot the `stock_price_high` column:

```
stock_data2.ix['CLI'].stock_price_high.plot()
```

The graph should look like:

![performing-data-analysis-with-python-16](https://user-images.githubusercontent.com/21102559/40942863-5b955f86-681d-11e8-8d24-953a8bbe032f.png)

### Result

You have used Python libraries to analyze the NYSE historical data in Python using DataFrames, and taking advantage of Python libraries.

You are finished!
