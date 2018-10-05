## Defining a Pig User Defined Function in Python

### About This Lab

**Objective:** To write a Pig User Defined Function (UDF) in Python to allow Python code to execute on a Hadoop cluster

**Successful outcome:** Your Pig script will invoke a Python UDF that computes the frequency distribution of a dataset of emails and outputs the top 5 words found in those emails

**File locations:** `~/labs`

---
### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>1. Open the UDFs Notebook</h2>

1\. Open your Web browser to the IPython Notebook/Jupyter.

2\.  From the `Dashboard` page, navigate into the `labs` folder and click on the `UDFs` notebook.

It should look like the following:

![defining-a-pig-user-defined-function-in-python-1](https://user-images.githubusercontent.com/21102559/40942631-d88e66d2-681c-11e8-87c7-9bcb7c07ae7e.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>2. Import the Modules</h2>

1\.  You will be using the replace function from string, and the `parseaddr` function from `email.utils`.

After the first line of code, and before the `for_split` list definition, add the following two imports:

```
import string
from email.utils import parseaddr
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>3. Define the `getFromEmail` Function</h2>

1\.  In this step, you will write a Python function that takes in the "From" field of an email and parses out the email address.


The return value for Pig is a `chararray`, so start by adding the following line of code after the `ignored` list definition:

```
@outputSchema("fromEmail:chararray")
```

2\.  Name the method called `getFromEmail`:

```
def getFromEmail(fromEmail):
```

3\.  The `parseaddr` function does all the work we need, so your function is only one line of code:

```
return parseaddr(fromEmail)[1]
```

4\.  Your Notebook should look like the following:

![defining-a-pig-user-defined-function-in-python-2](https://user-images.githubusercontent.com/21102559/40942632-d89bd736-681c-11e8-9365-0f3db800e451.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>4. Write the `getTop5Words` Function</h2>

1\.  In this step, you will write a Python function that returns the top five words found in the body of a collection of emails.

The return value for Pig is going to be a bag of tuples, so add the following `@outputSchema` declaration at the end of the cell after your `getFromEmail` function:

```
@outputSchema("y:bag{t:tuple(word:chararray, wordcount:int)}")
```

2\.  Name the function getTop5Words and call the parameter bag:

```
def getTop5Words(bag):
```

3\.  Inside the function, define a few local variables:

```
result = [] 
i = 0
wordcount = {}
i=0 
wordcount = {} 
```

4\.  The data passed to `bag` is a collection of sentences. You are going to replace all the punctuation in the sentences with a space character.


Add the following to the function definition:

```
for record in bag: 
	doc = record[0]
	for ch in for_split:
	     doc = string.replace(doc, ch, ' ')
```

5\. The `doc` value now contains a collection of words with all punctuation removed.

Add the following `for` loop (at the same level as the `for ch in for_split` loop), which splits the collection into individual words, and then keeps a running sum of the total occurrences of each word in a dict named `wordcount`:

```
for word in [w.lower() for w in string.split(doc)]: 
     if word not in wordcount:
          wordcount[word] = 1 
     else:
          wordcount[word] += 1
```

6\.  At this point in your script, the `wordcount` array contains the total count of every word. We just need to locate the top 5 occurring words, which is easily done by sorting the array.

Add the following for loop (at the same level as for record in bag) that creates a new tuple named result and populates it with the first 5 words in the sorted wordcount array:

```
for word in [w.lower() 
	for w in string.split(doc)]: 
	if word not in wordcount:
		wordcount[word] = 1
	else:
		wordcount[word] += 1
```

7.	At this point in your script, the wordcount array contains the total count of every word. We just need to locate the top 5 occurring words, which is easily done by sorting the array.

Add the following `for` loop (at the same level as `for record in bag`) that creates a new tuple named `result` and populates it with the first 5 words in the sorted wordcount array:

```
for word in sorted(wordcount, key=wordcount.get, reverse=True): 
     if not word in ignored and i < 5 and wordcount[word] > 1:
	  tup = (word, wordcount[word]) 
	  result.append(tup)
	  i=i+1
```

7\.  At the end of the function (at the same level as `for record in bag`) return the result:

```
return result
```

8\.  Save your `UDFs` Notebook.

9\.  The code should look like the following:

![defining-a-pig-user-defined-function-in-python-3](https://user-images.githubusercontent.com/21102559/40942633-d8ab0346-681c-11e8-81d2-20d1925a4264.png)



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>5. Test the Code Locally</h2>

1\.  Try running the code in the cell of your UDFs Notebook. You will get a compiler error because `outputSchema` is not defined.
Comment out these two lines of code and run the cell to verify that your Python code's syntax is valid.

2\.  Now uncomment the two `@outputSchema` lines of code.

> Tip: To test your Python UDFs locally without having to comment out the `outputSchema` lines, add the following code to the top of your script:

```
if   name   != '  lib  ':  
	def outputSchema(empty):
		return lambda x: x
```

This defines `outputSchema` as nothing when running locally, but the if statement will be `false` when running within a Pig script.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>6. Copy the Code to the VM</h2>

> Note To change which directory Firefox saves files, press `alt+f` to display the Firefox menus.

***Edit-\> Preferences -\> Browse***

1\.  From the IPython menu, select ***File -\> Download as -\> Python*** (`.py`):

2\.  Save the file as `udfs.py` in the `/[user-name]/labs/` folder.

![defining-a-pig-user-defined-function-in-python-4](https://user-images.githubusercontent.com/21102559/40942634-d8b9acd4-681c-11e8-896a-28be316081f2.png)

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>7. Put the Data in HDFS</h2>

1\.  Open a Terminal window on your VM

2\.  Put the file `mbox7.avro` into HDFS:

```
# hdfs dfs -put mbox7.avro
```

3\.  Verify the file is in HDFS:

```
hdfs dfs -ls
```

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo">
<h2>8. Invoke the UDF from Pig</h2>

1\.  Run the `pig` program from the `Lab7.1` folder to enter the `Grunt` shell.

2\. You are going to write a series of Pig commands that loads the data in `mbox7.avro` and invokes the Python UDF in `udfs.py`.

Start by registering the functions:

```
grunt> register 'udfs.py' using jython as udfs;
```

You should see your two functions registered: `getTop5Words` and `getFromEmail`:

```
[MainThread] INFO	org.apache.pig.scripting.jython.JythonScriptEngine - Register scripting UDF: udfs.getTop5Words
[MainThread] INFO	org.apache.pig.scripting.jython.JythonScriptEngine - Register scripting UDF: udfs.getFromEmail
```

3\.  Load the data in `mbox7.avro` in HDFS into a relation named `emails`:

```
grunt> emails = load 'mbox7.avro' using AvroStorage();
```

4\.  Use `describe` to view the schema of the Avro-formatted data:

```
grunt> describe emails
```

The schema should look like:

```
emails: {From: chararray,Subject: chararray,Date: chararray,Content: chararray}
```

5\.  Group the emails by the sender of the email:

```
grunt> emailsBySender = group emails by udfs.getFromEmail(From) parallel 3;
```

6\.  What effect will the "parallel 3" clause have on this job when it executes?

7\.  Define a projection that generates the email sender and the top 5 words in the body of the emails by invoking the `getTop5Words` UDF:

```
grunt> senderWords = foreach emailsBySender generate 
     group as fromEmail,
     udfs.getTop5Words(emails.(Content)) as top5Words;
```

8\.  Dump the `senderWords` relation to run the code on your Hadoop cluste and display the results:

```
grunt> dump senderWords;
```

9\.  Wait for the MapReduce job to execute. If successful, the tail of the output should look like:

```
(Kumar_Chinnakali@infosys.com,{(e,40),(mail,36),(this,24),(or,20),(any,16)}) 
(bharati.adkar@mparallelo.com,{(09,141),(13/07/16,140),(info,124),(53,107),(j ava,87)}) 
(hivehadooplearning@gmail.com,{(java,35),(at,28),(1,24),(hive,24),(job,21)})
(j.barrett.strausser@gmail.com,{(rank,131),(by,90),(1,84),(is,77),(function,6 1)})
(hivehadooplearning@gmail.com,{(java,35),(at,28),(1,24),(hive,24),(job,21)}) 
(j.barrett.strausser@gmail.com,{(rank,131),(by,90),(1,84),(is,77),(function,6 1)}) 
(Peter.Marron@trilliumsoftware.com,{(0,10),(this,8),(is,8),(type,8),(can,8)}) 
(ramasubramanian.narayanan@gmail.com,{(field,24),(addr,22),(emp_id,13),(row_c reate_date,12),(mast,12)})
```

### Result

You have written a Pig UDF in Python that finds the top 5 words in a collection of emails. This lab demonstrates how to take advantage of the power and usefulness of Python in combination with Hadoop using Pig. This is a great technique for integrating the myriad of Python libraries and capabilities with Hadoop!

You are finished!
