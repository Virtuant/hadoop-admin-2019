## Exploring Data with Apache Pig

### About This Lab

**Objective:** To use Apache Pig to navigate HDFS and explore a dataset

**Successful outcome:** You will have written several Pig scripts that analyze and query the White House visitors' data, including a list of people who visited the President

**File locations:** `whitehouse/visits.txt` in HDFS


---
### Steps


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>1. Load the White House Visitor Data</h2>

1\.  You will use the TextLoader to load the `visits.txt` file. Define the following `LOAD` relation:

```
grunt> A = LOAD '/user/[user-name]/whitehouse/' USING TextLoader();
```

<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>2. Count the Number of Lines</h2>

1\. Define a new relation named `C` that is a group of all the records in `A`:

```
grunt> C = GROUP A ALL;
```

2\. Use `DESCRIBE` to view the schema of `C`. What is the datatype of the group field?

3\. Where did this data type come from?

4\.  Why does the `A` field of `C` contain no schema?

5\.  How many groups are in the relation `C`?

6\.  The `A` field of the `C` tuple is a bag of all of the records in `visits.txt`. Use the `COUNT` function on this bag to determine how many lines of text are in visits.txt:

```
grunt> A_count = FOREACH C GENERATE 'rowcount', COUNT(A);
```

> Note  The `rowcount` string in the `FOREACH` statement is simply to demonstrate that you can have constant values in a GENERATE clause. It is certainly not necessary - it just makes the output nicer to read.

7\. Use `DUMP` on `A_count` to view the results. The output should look like: 

```
(rowcount,447598)
```  

We can now conclude that there are `447,598` rows of text in `visits.txt`.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>3. Analyze the Data's Contents</h2>

1\.  We now know how many records are in the data, but we still do not have a clear picture of what the records look like.

Let's start by looking at the fields of each record. Load the datausing `PigStorage(',')` instead of `TextLoader()`:

This will split up the fields by comma.

```
grunt> visits = LOAD '/user/[user-name]/whitehouse/' USING PigStorage(',');
```

2\. Use a `FOREACH..GENERATE` command to define a relation that is a projection of the first 10 fields of the visits relation.

3\. Use `LIMIT` to display only 50 records, then `DUMP` the result. The output should be 50 tuples that represent the first 10 fields of visits:
```
(PARK,ANNE,C,U51510,0,VA,10/24/2010 14:53,B0402,,) 
(PARK,RYAN,C,U51510,0,VA,10/24/2010 14:53,B0402,,) 
(PARK,MAGGIE,E,U51510,0,VA,10/24/2010 14:53,B0402,,) 
(PARK,SIDNEY,R,U51510,0,VA,10/24/2010 14:53,B0402,,) 
(RYAN,MARGUERITE,,U82926,0,VA,2/13/2011 17:14,B0402,,) 
(WILE,DAVID,J,U44328,,VA,,,,)
(YANG,EILENE,D,U82921,,VA,,,,)
(ADAMS,SCHUYLER,N,U51772,,VA,,,,)
(ADAMS,CHRISTINE,M,U51772,,VA,,,,) 
(BERRY,STACEY,,U49494,79029,VA,10/15/2010 12:24,D0101,10/15/2010 14:06,D1S)
```

> Note  Because `LIMIT` uses an arbitrary sample of the data, your output will be have different names, but the format should look similar.

---
> Notice from the output that the first three fields are the person's name. The next 7 fields are a unique ID, badge number, access type, time of arrival, post of arrival, time of departure and post of departure.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>4. Locate the POTUS (President of the United States of America)</h2>

1\.  There are 26 fields in each record, and one of them represents the *visitee* (the person being visited in the White House).

Your goal now is to locate this column and determine who has visited the President of the United States.

Define a relation that is a projection of the last 7 fields (`$19` to `$25`) of visits. Use `LIMIT` to only output 500 records. The output should look like:
```
(OFFICE,VISITORS,WH,RESIDENCE,OFFICE,VISITORS,HOLIDAY OPEN HOUSE/) 
(OFFICE,VISITORS,WH,RESIDENCE,OFFICE,VISITORS,HOLIDAY OPEN HOUSES/) 
(OFFICE,VISITORS,WH,RESIDENCE,OFFICE,VISITORS,HOLIDAY OPEN HOUSE/) 
(CARNEY,FRANCIS,WH,WW,ALAM,SYED,WW TOUR) 
(CARNEY,FRANCIS,WH,WW,ALAM,SYED,WW TOUR) 
(CARNEY,FRANCIS,WH,WW,ALAM,SYED,WW TOUR) 
(CHANDLER,DANIEL,NEOB,6104,AGCAOILI,KARL)
```

It is not necessarily obvious from the output, but field `$19` in the `visits` relation represents the person being visited. Even though you selected 500 records in the previous step, you may or may not see `POTUS` in the output above. (The White House has thousands of visitors each day, but only a few meet the President!)


2\. Use `FILTER` to define a relation that only contains records of visits where field $19 matches `POTUS`. Limit the output to 500 records. The output should include only visitors who met with the President. For example:
```
(ARGOW,KEITH,A,U83268,,VA,,,,,2/14/2011 18:42,2/16/2011 16:00,2/16/2011 23:59,,154,LC,WIN,2/14/2011 18:42,LC,POTUS,,WH,EAST ROOM, THOMPSON,MARGRETTE,,AMERICA'S GREAT OUTDOORS ROLLOUT EVENT ,5/27/2011,,,,,,,,,,,,,,,,,,,,,,,,,) (AYERS,JOHNATHAN,T,U84307,,VA,,,,,2/18/2011 19:11,2/25/2011 17:00,2/25/2011 23:59,,619,SL,WIN,2/18/2011 19:11,SL,POTUS,,WH,STATE FLOO, GALLAGHER,CLARE,,RECEPTION ,5/27/2011,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,)
```



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>5. Count the POTUS Visitors</h2>

1\.  Let's discover how many people have visited the President. To do this, we need to count the number of records in visits where field $19 matches `POTUS`. See if you can write a Pig script to accomplish this. Use the potus relation from the previous step as a starting point. You will need to use `GROUP ALL`, and then a `FOREACH` projection that uses the `COUNT` function.

2\.  If successful, you should get `21,819` as the number of visitors to the White House who visited the President.




<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>6. Finding People Who Visited the President</h2>

1\.  So far you have used DUMP to view the results of your Pig scripts. In this step, you will save the output to a file using the `STORE` command. Start by loading the data using `PigStorage(',')`, which you may already have defined:

```
grunt> visits = LOAD '/user/[user-name]/whitehouse/' USING PigStorage(',');
```

2\. Now `FILTER` the relation by visitors who met with the President:

> Note this assumes the field desired is #29

```
potus = FILTER visits BY $19 MATCHES 'POTUS';
```

3\. Define a projection of the `potus` relationship that contains the name and time of arrival of the visitor:
```
grunt> potus_details = FOREACH potus GENERATE 
(chararray) $0 AS lname,
(chararray) $1 AS fname,
(chararray) $6 AS arrival_time,
(chararray) $19 AS visitee;
```

4\.  Order the `potus_details` projection by last name:
```
grunt> potus_details_ordered = ORDER potus_details BY lname;
```

5\.  Store the records of `potus_details_ordered` into a folder named `potus` and using a comma delimiter:
```
grunt> STORE potus_details_ordered INTO 'potus' USING PigStorage(',');
```

6\.  View the contents of the potus folder:
```
grunt> ls potus 

hdfs://sandbox.hortonworks.com:8020/user/[user-name]/potus/_SUCCESS<r 3>0 
hdfs://sandbox.hortonworks.com:8020/user/[user-name]/potus/part-r-00000<r 3>501378
```

7\. Notice there is a single output file, so the Pig job was executed with one reducer. 
View the contents of the output file using `cat`:
```
grunt> cat potus/part-r-00000
```

The output should be in a comma-delimited format and contain the last name, first name, time of arrival (if available), and the string `POTUS`:
```
CLINTON,WILLIAM,,POTUS 
CLINTON,HILLARY,,POTUS 
CLINTON,HILLARY,,POTUS 
CLINTON,HILLARY,,POTUS 
CLONAN,JEANETTE,,POTUS 
CLOOBECK,STEPHEN,,POTUS 
CLOOBECK,CHANTAL,,POTUS 
CLOOBECK,STEPHEN,,POTUS 
CLOONEY,GEORGE,10/12/2010 14:47,POTUS
```



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h2>7. View the Pig Log Files</h2>

1\.  Each time you executed a `DUMP` or `STORE` command, a MapReduce job executed on your cluster. 
You can view the log files of these jobs in the JobHistory UI. 
Change your browser ***Ambari -> MapReduce2 -> Quick Links -> JobHistory UI***

![exploring-data-with-apache-pig-1](https://user-images.githubusercontent.com/21102559/40942661-ec3cfa86-681c-11e8-8372-874f8e0c8e04.png)

2\. Click on the job's id to view the details of the job and its log files.


### Result

You have written several Pig scripts to analyze and query the data in the White House visitors' log. You should now be comfortable with writing Pig scripts with the Grunt shell and using common Pig commands like `LOAD`, `GROUP`, `FOREACH`, `FILTER`, `LIMIT`, `DUMP` and `STORE`.

### Answers

**Step 2:** The group field is a chararray because it is just the string "all" and is a result of performing a `GROUP ALL`. The A field of `C` contains no schema because the A relation has no schema. The `C` relation can only contain 1 group because it a grouping of every single record.

You are finished!

> Note  The A field is a bag, and A will contain any number of tuples.

#### Solutions

##### Step 3:

```
visits = LOAD '/user/[user-name]/whitehouse/' USING PigStorage(','); 
firstten = FOREACH visits GENERATE $0..$9;
firstten_limit = LIMIT firstten 50; 
DUMP firstten_limit;
```

##### Step 4:

```
lastfields = FOREACH visits GENERATE $19..$25; 
lastfields_limit = LIMIT lastfields 500;
DUMP lastfields_limit;

--find the POTUS
potus = FILTER visits BY $19 MATCHES 'POTUS'; 
potus_limit = LIMIT potus 500;
DUMP potus_limit;
```

##### Step 5:

```
potus = FILTER visits BY $19 MATCHES 'POTUS'; 
potus_group = GROUP potus ALL;
potus_count = FOREACH potus_group GENERATE COUNT(potus); 
DUMP potus_count;
```
