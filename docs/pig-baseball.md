## Pig Use Case: Baseball Stats

### Goals

* You will be given a fairly large set of game statistics (over 97,800 rows)
* Stats for each American baseball player by year from 1871
* Identify players who scored highest runs for each year in ascending order
* Determine First and Last name for the each player by joining 2 data sets
* You should try to accomplish this with minimal instruction

**Data Files:** Sean Lahman's baseball archive

----

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Review and understand Baseball statistics data files</h4>

In your VM, create a new directory in your home directory to use for sample data.

Via a browser, get the latest baseball stats file from Sean Lahman’s baseball stats web site (the
file version may change.) On a page [here](http://www.seanlahman.com/baseball-archive/statistics) you’ll see
the link for downloading some comma-delimited CSV files in a zipped archive. Get that archive (easiest
using your VM browser), expand it and do a put of the files needed into HDFS.

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) you can also achieve this from a Linux shell like this:

```
$ wget https://github.com/chadwickbureau/baseballdatabank/archive/v2019.2.zip
```

(the latest file as of January 2016 – zip file name may change)

Unzip the file in your new directory</h4>
  
Many statistics files will unpack from the file.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Put Batting.csv into HDFS</h4>

|# |NAME |DESCRIPTION|
|---|---|---|
|0 |playerID|A unique code assigned to each player. The playerID links the data in this file with records in other files|
|1 |yearID |Year|
|2 |stint |player's stint (order of appearances in season)|
|3 |teamID |Team|
|4 |lgID |League|
|5 |G |Games|
|6 |AB |At Bats|
|7 |R |Runs|
|8 |H |Hits|
|9 |2B |Doubles|
|10 |3B |Triples|
|11 |HR |Homeruns|
|12 |RBI |Runs Batted In|
|13 |SB |Stolen Bases|
|14 |CS |Caught Stealing|
|15 |BB |Base on Balls|
|16 |SO |Strikeouts|
|17 |IBB |Intentional walks|
|19 |HBP |Hit by pitch|
|19 |SH |Sacrifice hits|
|20 |SF |Sacrifice flies|
|21 |GIDP |Grounded into double plays|

Consider following fields/columns</h4>
  
* Column # 0 (Player ID)
* Column # 1 (Year)
* Column # 7 (Runs)

Put the file `Batting.csv` into HDFS under an appropriate directory, such as `batting/input`.

You may wish to rename the file in HDFS. Use `hdfs dfs –cat` to verify the put if needed.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Do the same with People.csv</h4>

|# |NAME |DESCRIPTION|
|0 |playerID |Unique ID for player|
|1 |birthYear |Year player was born|
|13 |nameFirst |Player's first name|
|14 |nameLast |Player's last name|
|15 |nameGiven |Player's given name (typically first and middle)|
|||22 more fields|

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Identify players who scored highest runs for each year</h4>

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) If you haven’t yet, create a new directory, such as `baseball/input` in HDFS and put statistic
files Batting.csv & Master.csv into this directory from your local machine. You may rename them in
HDFS as you wish.

For this lab, you should consider following fields/columns:

#### Batting.csv
* Column # 0 (Player ID)
* Column # 1 (Year)
* Column # 7 (Runs)

#### People.csv
* Column # 0 (Player ID)
* Column # 13 (First Name)
* Column # 14 (Last Name)

Start the Pig shell.

Load batting data into Pig using the PigStorage() function.

![note](https://user-images.githubusercontent.com/558905/40528492-37597500-5fbf-11e8-96a1-f4d206df64ab.png) The default delimiter in Pig is a TAB (\t). A CSV file has comma-separated data in each
line, so we need to inform Pig explicitly about that field delimiter. See [here](http://pig.apache.org/docs/r0.16.0/basic.html#load) for the correct
function syntax.

Read relevant fields from the loaded data. In this case we are interested in 1st , 2nd and 9th fields
for each record. Use FOREACH-GENERATE statements to accomplish this task. Look [here](http://pig.apache.org/docs/r0.16.0/basic.html#foreach) for correct
FOREACH syntax.

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) In Pig Latin $0 can represent the 1st field, $1 the 2nd field and so on. In the GENERATE
statement, you can use the “$X AS (alias:type)” syntax to create text aliases for positional fields.
More on this [here](http://pig.apache.org/docs/r0.16.0/basic.html#expressions).

Group runs from step 4 by year. See the syntax for GROUP [here](http://pig.apache.org/docs/r0.16.0/basic.html#group).

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png)  use DUMP and DESCRIBE to validate your assumptions along the way

Use FOREACH-GENERATE, GROUP and MAX functions in Step 5 data to get max runs for
each year.

Now, join Step 4 and Step 6 data based on the ‘year’ and ‘runs’ fields. Inner and outer joins
syntax is shown here.

To identify the playerID who scored the highest one for each year, create a new bag with Year,
PlayerID and Max Run data using FOREACH, GENERATE on Step7 data.

Check the output of the above exercise using a DUMP command.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>5. Determine First and Last name for the each player</h4>

Load master data to Pig using the PigStorage() function.

Read relevant fields from the file. In this case we are interested in 13th & 14th fields for each
record using FOREACH and generate command:

Join PLAYERS dataset with the result dataset from previous task (Step 9) based on the common
field ‘playerID’:

Create a new dataset having Year, Player’s First and Last Name and the MAX from Step 3 using
FOREACH and GENERATE commands.

Make sure that data in Step 4 is sorted on Year in ascending order

Get the output of Step 5 using DUMP command

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>[OPTIONAL] Run a similar job using Pig Streaming</h4>

The syntax for Pig streaming is here.

* Write a simple script to take stdin and massage to stdout
* In Pig, LOAD data in using PigStorage() syntax.
* Use DEFINE to reference your script in Pig query to come up with similar results as before.

### Results

Comgrats! Finished team score 100 to 0!!!
