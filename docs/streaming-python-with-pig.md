## Streaming Python with Pig

### About This Lab

**Objective:** To write a Pig script that streams data through a Python script

**Successful outcome:** A frequency distribution of the words in a dataset

**File locations:** `~/labs`

---
Steps
---------


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. Create a Notebook</h2>

1\.  Create a new Notebook in IPython Notebook/Jupyter

2\.  Change the name of the Notebook to `freqdist`


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Import the Modules</h2>

1\.  Start by adding the following to the beginning of your script:
```
#!/usr/local/bin/python2.7
```

2\.  Import the `fileinput` module.

3\.  Import the `string` module.

4\.  You are going to use the `FreqDist` function in the `nltk` module to determine the frequency of words in a dataset of github commits.

Import this function:

```
from nltk import FreqDist
```

5\.  You also need the `RegexpTokenizer` function:

```
from nltk.tokenize import RegexpTokenizer
```

Your Notebook should look like:

![streaming-python-with-pig-1](https://user-images.githubusercontent.com/21102559/40942914-8371fe9c-681d-11e8-9eab-2cbb5326ffeb.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Clean the Input Data</h2>

1\.  The `string` module contains a `punctuation` object that contains common punctuation characters.

We are going to use this object to strip out common punctuation from our input data. 

Start by defining the following variable:

```
replace_punctuation = string.maketrans(string.punctuation, ' '*len(string.punctuation))
```

<blockquote>
<b>Reference</b><br>
For "maketrans" and "translate" see:<br>
    <a>https://docs.python.org/2/library/string.html</a>
</blockquote>

`replace_puncutation` is a tuple of punctuation and `x`


2\.  Declare a new empty string to represent the cleaned data:

```
cleaned_input = ""
```

3\.  Add the following `for` loop, which iterates through the input text, strips out any leading or trailing whitespace, and creates a string object named `lines` that contains all of the cleaned input data:

```
for line in fileinput.input(): 
     line=line.strip()
     line=line.translate(replace_punctuation) 
     cleaned_input += line + " "
```

> Reference For more on fileinput see: <https://docs.python.org/2/library/fileinput.html>


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Compute the Frequency Distribution</h2>

1\.  We are going to parse the input text into words using a `RegexpTokenizer` object.

Add the following line of code (outside of the above `for` loop):

```
tokenizer = RegexpTokenizer(r'\w+')
```

> Note The `r 'w+'` is a Python regular expression that represents a word of text, so this particular tokenizer will split text into words. <http://www.nltk.org/_modules/nltk/tokenize/regexp.html>

2\.  Compute the frequency distribution:

```
freq = FreqDist(tokenizer.tokenize(cleaned_input))
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>5. Output Key/Value Pairs</h2>

1\.  This script is being used in a MapReduce job. Whatever text is output from this script becomes the `<key,value>` pairs output by the corresponding mapper or reducer.

Add the following `for` loop that iterates through the frequency distribution and outputs the top 50 occurring words:

```
for k, f in freq.most_common(50): 
    print "%s\t%s" % (k, f)
 
```

Your script should look like the following:

![streaming-python-with-pig-2](https://user-images.githubusercontent.com/21102559/40942916-83827024-681d-11e8-856d-4ffa68f810ba.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h1>6. Save the Script</h1>

> **Warning** "Running" this code in IPython results in an error because there is no given input for `fileinput.input()`



1/.  Save the Notebook as `freqdist.py` in your `/[user-name]/labs` folder of your VM. 


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>7. Put the Data in HDFS</h2>

<!-- -->

1\.  From a Terminal window, change directories to the `~/labs` dir


2\.  Put the input file into HDFS:

```
hdfs dfs -put commits.json
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>8. Test the Code Locally</h2>

1\.  Before running the `freqdist.py` code on your cluster, it is a good idea to make sure the code runs successfully.

Enter the following command:

```
echo "This is a test" | python .freqdist.py
```

2\.  Look through the output. If your code has any typos or errors, fit them in the Notebook and then re-save the Notebook as `freqdist.py`.

Keep fixing any bugs or typos until the command above runs successfully.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>9. Ship the Python Code</h2>

1\.  Start the Grunt shell:

```
pig
```

2\.  The first step is to use the `ship` command to ensure your Python script is available on the NodeManagers that will be executing the code.

> Note  Those are backticks around `python freqdist.py`, and regular single quotes in the call to ship:

```
define freqdist `freqdist.py` ship ('freqdist.py');
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>10. Register the Necessary JARs</h2>

1\.  The data being processed is in a JSON format that requires the following JAR files (which are in the `labs` folder):

```
register 'elephant-bird-pig-4.4.jar';
register 'elephant-bird-hadoop-compat-4.4.jar'; 
register 'json-simple-1.1.jar'; 
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>11. Load the Input File</h2>

1\.  Load `commits.json` into a relation called `commits` using the following `JsonLoader`:]

```
commits = load 'commits.json' USING com.twitter.elephantbird.pig.load.JsonLoader('-nestedLoad') as (json:map[]); 
```

2\.  Notice the data is loaded as a `map`:

```
grunt> describe commits 
commits: {json: map[]} 
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>12. Transform the Data</h2>

1\.  We only want the "message" in the input data, so define a projection that only contains that field:

```
messages = foreach commits generate json#'commit'#'message';
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>13. Compute the Frequency Distribution</h2>

1\.  Stream the data through your Python code:

```
analyzed = stream messages through freqdist;
```

2\.  We only want the word and its number of occurrences from the result:

```
structured_analysis = foreach analyzed generate (chararray) $0 as word:chararray, (int) $1 as count:int;
```

3\.  View the result:

```
dump structured_analysis;
```

The output should look similar to:

```
(the,42) 
(to,42) 
(from,38) 
(Merge,37) 
(request,37) 
(pull,35) 
(for,30) 
(ci,27)
(skip,27) 
...
```

### Result

You have written a Python script that gets invoked within a Pig script using Hadoop streaming.

You are finished!
