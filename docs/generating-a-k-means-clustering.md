## Generating a K-Means Clustering

### About This Lab

**Objective:** To use Scikit-Learn to generate a `k-means` clustering of a collection of text documents

**Successful outcome:** A graph showing the `k-means` cluster of a group of documents that represent newsgroup postings

**Files Location** `~/labs`

---
### Steps

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1. Navigate to the directory where the input Avro file is located</h2>
a. Make sure you are in the appropriate directory before continuing.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Put the Data in HDFS</h2>

1\.  Put the `newsgroup.avro` file into the root user's home folder in HDFS:
```
hdfs dfs -put newsgroups.avro
```

2\.  Verify the file is in HDFS:
```
hdfs dfs -ls /user/[user-name]
```


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. Compute the Cluster Centers</h2>

1\.  In IPython, open the kmeans notebook in the `labs` folder.

2\.  Notice the modules have been imported for you in the first cell.

3\.  Notice the code in the third cell processes the input data by newsgroup topic, invoking the `run_kmeans` function (which is not completely written yet in the second cell).

4\.  In the second cell is a partially written function named `run_kmeans`. Where the `TODO` comment mentions creating a KMeans object, add the following line of code:

```
#TODO: Create a KMeans object
km = KMeans(n_clusters=num_clusters, init='k-means++',
     max_iter=max_iterations, n_init=num_seeds, verbose=0)
```

5\.  In the `TODO` comment mentioning computing cluster centers, invoke the `fit_predict` function of `KMeans`, passing in the vectorized text:
```
# TODO: Compute cluster centers and predict cluster index for each newsgroup topic
clusters = km.fit_predict(feature_vectors)
```

6\.  Return the result in a new DataFrame using the following code:
```
#TODO: Return the results in a new DataFrame
result = DataFrame(dataset, columns = ["topic", "article_id"]) 
result["cluster_id"] = clusters
return result
```

7\.  Save your changes to the `kmeans` notebook. 



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>4. Save `kmeans.py` as a File</h2>

1\. From IPython, select **File -\> Download as -\> Python** (`.py`). Save the file as `kmeans.py` into the `~/labs` folder.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>5. Write the Pig Script</h2>

1\.  Most of the Pig script is written for you. From the `/[user-name]/labs` folder in HDFS, open the script with the `gedit` text editor:
```
gedit kmeans.pig
```

2\.  Notice the `newsgroups.avro` file is loaded from HDFS using the `AvroStorage` class.

3\.  The next section of code filters out the smaller data sets (those with fewer than 100 articles posted in them) and stores the results in the `news_by_major_topic` relation.

4\.  The next section of Pig code groups the data by topic, which is a critical step since we are running this code in a distributed fashion. Based on the code, how many reducers will the ensuing MapReduce job use?

5\.  At the end of the script, add the following stream command that streams the data through our `kmeans.py` script:
```
result = stream filtered_content through kmeans;
```

6\.  Transform the result by adding a name and data type to each field:
```
structured_result = foreach result generate (chararray) $0 as topic, (int) $1 as articleId, (int) $2 as clusterId;
```

7\.  Sort the results:
```
sorted = order structured_result by topic, clusterId, articleId;
```

8\.  Finally, store the results into a folder in HDFS:
```
store sorted into 'kmeans_output';
```

9\.  Save your changes to `kmeans.pig` and close `gedit`. 



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Run the Pig Script</h2>


1\.  From the Sandbox prompt, run the Pig script:
```
pig kmeans.pig
```

2\.  Wait for the job to run on your cluster, which may take a few minutes. 
    

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. View the Results</h2>


1\.  Verify the output files were created in the `kmeans_output` folder in HDFS:
```
hdfs dfs –ls kmeans_output
```
Output:
```
Found 2 items
-rw-r--r-- 3 root root 0        kmeans_output/_SUCCESS 
-rw-r--r-- 3 root root 460308   kmeans_output/part-r-00000
```

2\.  View the results file:
```
hdfs dfs -tail kmeans_output/part-r-00000
```

The output should look like a topic, a document name, and the cluster that the document was categorized in:

```
sci.space	62478	0
sci.space	62479	2
sci.space	62480	2
sci.space	62481	1
sci.space	62614	6
sci.space	62615	1
sci.space	62616	5
sci.space	62708	2
sci.space	62709	1

```

### Result

You have just taken a raw dataset of text, used Python to format it into a Pig-friendly format (Avro), then streamed the data through the Scikit-Learn K-Means clustering algorithm in a distributed manner on a Hadoop cluster.

You are finished!
