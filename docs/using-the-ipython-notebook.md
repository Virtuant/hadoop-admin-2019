## Using the IPython Notebook

### About This Lab

**Objective:** To write Python code using the IPython Notebook

**Successful outcome:** You will create a new Notebook and run some code, then open an existing Notebook that has you write some Python to process text

**File location:** `~/labs`

---
### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1. View the Notebook</h2>

1\.  Open your Web browser and point it to the IPython Notebook


2\.  You should see the Dashboard page of IPython Notebook. 

The latest version of IPython Notebook is called Jupyter, or at least at the time of writing this.

> Note  You can monitor the Jupyter service through Ambari as well.

![using-the-ipython-notebook-1](https://user-images.githubusercontent.com/21102559/40943104-06c132e0-681e-11e8-952d-3f094b9d51d3.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Create a New Notebook</h2>

1\.  Click the **New Notebook** button on the right side of the page.

Choose a **Python 2** notebook.

A new feature of Jupyter supports both R and Scala, as you can see.

2\. Notice a new tab opens in your Web browser with a new untitled notebook with a prompt for entering Python commands:

![using-the-ipython-notebook-2](https://user-images.githubusercontent.com/21102559/40943106-06cf8c78-681e-11e8-9fba-9ec842e24c65.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. Run Commands</h2>

1\.  Define a new variable named `x`, assign it to 20, and print it out:

![using-the-ipython-notebook-3](https://user-images.githubusercontent.com/21102559/40943107-06e2ff24-681e-11e8-950a-bcc3ef5edd5e.png)

---
![using-the-ipython-notebook-4](https://user-images.githubusercontent.com/21102559/40943108-06f31af8-681e-11e8-9adb-0ec019b9a431.png)

2\.  Click the "Run cell" button in the toolbar, which looks like a "Play" arrow. Your code in the cell should run, and 20 should be displayed:

![using-the-ipython-notebook-5](https://user-images.githubusercontent.com/21102559/40943110-070268be-681e-11e8-9839-674132bb9347.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>4. Save the Notebook</h2>

1\. To save a Notebook, click the `Disk` icon on the toolbar (the first button on the left). This saves the Notebook as `Untitled0`.

2\. To rename the Notebook, click on the `Untitled0` name at the top of the page. When the dialog window pops up, enter `Practice` for the Notebook name:

![using-the-ipython-notebook-6](https://user-images.githubusercontent.com/21102559/40943111-07152242-681e-11e8-8842-4b081dc8939f.png)

3\. Click `OK` and your new Notebook name should now appear at the top of the page.

4\.  Click the `Save` button again on the toolbar to save your changes. 



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>5. Close and Reopen the Notebook</h2>

1\. Close the tab in your Web browser that is displaying the `Practice` Notebook.

2\. Back on the `Dashboard` page, you should see `Practice` in the list of Notebooks.

3\. Click on `Practice` to reopen the Notebook in a new tab. Notice the Notebook looks just like it did when you closed it.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Define a List</h2>

1\. In your `Practice` Notebook, add the following line of code that defines a list named `mylist`:

```
mylist = \[-3, -2, -1, 0, 1, 2\]
```

2\.  Add a 3 to the end of the list:

```
mylist.append(3)
```

3\. Print `mylist` and run the code in this cell. The output should look like:

![using-the-ipython-notebook-7](https://user-images.githubusercontent.com/21102559/40943112-072b4068-681e-11e8-88ca-c880d9c836ec.png)

4\. Define a list comprehension named `cubes` that is the cube of each number in `mylist`: 

```
cubes = [x**3 for x in mylist]
```

5\. Print the `cubes` variable and verify it is defined correctly:

![using-the-ipython-notebook-8](https://user-images.githubusercontent.com/21102559/40943114-073d1464-681e-11e8-88b4-abd6ef763597.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. Define a Function</h2>

1\.  Define a new function called `myfunc` that represents a polynomial:
```
def myfunc(x):
      return 2*x**3 - 5*x**2 + 1
 
```

2\.  Use the range function to define a variable named `input` that represents integers from `-2` to `6`:

```
input = range(-2,6)
```

3\. Define a list comprehension named `y` that invokes `myfunc` for each value in `input`:

```
y = [myfunc(x) for x in input]
```

4\.  Print `y`, run the cell, and verify the output is:

```
[-35, -6, 1, -2, -3, 10, 49, 126]
```



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>8. Plot Values</h2>

1\.  Let's look at using a function from a library. Import the `pyplot` function from the `matplotlib` library:

```
import matplotlib.pyplot as plt
```

2\.  Set `y` (from the previous cell) as the values to be plotted:

```
plt.plot(y)
```

3\. Run the code in the cell. Notice the `pyplot` function interpolates a simple graph based on the input values:

![using-the-ipython-notebook-9](https://user-images.githubusercontent.com/21102559/40943115-074fce92-681e-11e8-92e0-422a25664d63.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>9. Open the TextDemo Notebook</h2>

1\. Select **File -\> Open** from the Notebook menu. The Dashboard page will appear.

2\.  Click on the **labs** folder.

3\.  Click on the **TextDemo.ipynb** Notebook. It will open in a new tab.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>10 . Working with Strings</h2>

1\. In the first cell, notice an array of strings is defined named `tweets`.

2\. Write a `for` loop that iterates through `tweets` and prints out each value of the list:

```
for tweet in tweets : 
     print tweet
```

3\. Run the code. The output should look like:

![using-the-ipython-notebook-10](https://user-images.githubusercontent.com/21102559/40943116-075c6756-681e-11e8-9647-5f22d959cd2b.png)

4\. Notice the second cell defines three lists:

```
positive_words = ['awesome', 'good', 'nice', 'super', 'fun'] 
negative_words = ['awful','lame','horrible','bad'] 
neutral_words = ['ok','fine','decent']
```

5\. Within this second cell, add a **for** loop that iterates through the **tweets** list:

```
for tweet in tweets :
```

6\. Inside this `for` loop, split each `tweet` into a list of words and the print the list:

```
words = tweet.split() 
print words 
```

7\.  Run the code. The output should look like:

![using-the-ipython-notebook-11](https://user-images.githubusercontent.com/21102559/40943117-076a1860-681e-11e8-96d6-999b6814a60a.png)

8\.  Comment out the line of code that prints the words list:

```
print words
```

9\. Inside the for loop, add another for loop that iterates through the words and prints out a word if it appears in the `negative_words` list:

```
for tweet in tweets : 
     words = tweet.split()
     
#   print words
    for word in words :
         if word in negative_words : 
              print tweet
```

10\.  Run the code in the cell. The output should be one tweet:

```
The food in the school cafeteria is awful
```

11\. Why did the tweet "The Yankees are so lame!" not display? 



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>11. Clean the Data</h2>


1\. The tweets contain punctuation and potentially capitalized letters that are affecting the logic of the "in" comparison. In this step, you will clean the text by making it lowercase and stripping out any punctuation. In a new cell, add the following import:

```
from string import punctuation
```

2\. To view what **punctuation** is defined as, print it out and run the code in the cell:

![using-the-ipython-notebook-12](https://user-images.githubusercontent.com/21102559/40943118-0775ef96-681e-11e8-81d4-9aba64a74696.png)

3\. In a new cell, add the following nested **for** loop:

```
for tweet in tweets : 
words = tweet.split() 
for word in words : 
```

4\. Within the nested `for` loop, convert the word to lowercase:

```
word_lowered = word.lower()
```

5\. Strip out any punctuation of the lowercase word:

```
word_clean = word_lowered.strip(punctuation)
```

6\. If the cleaned word is in the `negative_words` list, print out the tweet:

```
if word_clean in negative_words : 
    print tweet
 
```

7\.  Run the code in the cell. You should see two negative tweets this time:

![using-the-ipython-notebook-13](https://user-images.githubusercontent.com/21102559/40943119-078c8a3a-681e-11e8-808b-26c03735c701.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>12. Use the Slice Notation</h2>

1\.  In a new cell, add the following **for** loop:

```
for tweet in tweets :
```

2\. Within the for loop, add the following print command, which prints the first 10 characters of each tweet:

```
print tweet\[0:9\]
```

3\. Run the cell. The output should look like:

![using-the-ipython-notebook-14](https://user-images.githubusercontent.com/21102559/40943120-079e04a4-681e-11e8-9e09-ac0f3eff3d78.png)

4\. Play around with the slice feature. For example: 

Determine what [:15] and [15:] display.

What does [-10:] display?

What does [-10:-5] display?

Try [5:len(tweet)] and [:-1].



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>13. Counting Tweets</h2>

1\.  In this step, we will not give you any help. Write code that prints out the number of tweets in the `tweets` list that contain a word from `positive_words`.

### Result

You have written a fair amount of Python code using IPython Notebook. Hopefully at this point you are comfortable with the Python syntax and have a good starting foundation for the language. We will use Python throughout the remainder of the course, running code both in Notebook and on a Hadoop cluster.

You are finished!
