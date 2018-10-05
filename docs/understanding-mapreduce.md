## Understanding MapReduce

### About this Lab

**Objective:** To understand how MapReduce works.

**During this Lab:** Watch as your instructor performs the following steps.

**Related lesson:** The MapReduce Framework

**Lab Files** `~/labs`

---
### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>1. Put the File into HDFS</h2>


1\. Use more to look at a file named `constitution.txt`. Press `q` to exit when finished.

```
more constitution.txt
```

2\. Put the file into HDFS:

```
hdfs dfs -put constitution.txt
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>2. Run the WordCount Job</h2>

1\.  The following command runs a wordcount job on the `constitution.txt` and writes the output to `wordcount/output`:

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount constitution.txt wordcount_output
```

> Note  that a MapReduce job gets submitted to the cluster. ***Wait for the job to complete***.
    

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>3. View the Results</h2>

1\.  View the contents of the `wordcount_output` folder:

```
hdfs dfs -ls wordcount_output
```

Output:

```
Found 2 items
-rw-r--r-- 1    root    root    0            wordcount_output/_SUCCESS 
-rw-r--r-- 1    root    root    17049        wordcount_output/part-r-00000
```

2\.  Why is there one `part-r` file in this directory?

> ***Answer:*** The job only used one reducer.

3\.  What does the "r" in the filename stand for?

> ***Answer:*** The "r" stands for "reducer."

4\.  View the contents of `part-r-00000`:

```
hdfs dfs -cat wordcount_output/part-r-00000
```

5\.  Why are the words sorted alphabetically?

> ***Answer:*** The key in this MapReduce job is the word, and keys are sorted during the shuffle/sort phase.

6\.  What was the key output by the WordCount reducer?

> ***Answer:*** The reducer's output key was each word.

7\.  What was the value output by the WordCount reducer?

> ***Answer:*** The value output by the reducer was the sum of the 1's, which is the number of occurrences of the word in the document.

8\.  Based on the output of the reducer, what do you think the mapper output would be as `key/value` pairs?
    
> ***Answer:*** The mapper outputs each word as a key and the number 1 as each value.

### Result

You are finished!
