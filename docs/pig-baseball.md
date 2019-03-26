## Pig Use Case: Baseball Stats

### Goals:
• You will be given a fairly large set of game statistics (over 97,800 rows)
• Stats for each American baseball player by year from 1871-2013
• Identify players who scored highest runs for each year in ascending order
• Determine First and Last name for the each player by joining 2 data sets
• You should try to accomplish this with minimal instruction

----

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Review and understand Baseball statistics data files

1. In your VM, create a new directory in your home directory to use for sample data.

2. Via a browser, get the latest baseball stats file from Sean Lahman’s baseball stats web site (the
file version may change.) On the page http://www.seanlahman.com/baseball-archive/statistics you’ll see
the link for downloading some comma-delimited CSV files in a zipped archive. Get that archive (easiest
using your VM browser), expand it and do a put of the files needed into HDFS.

NOTE: you can also achieve this from a Linux shell like this:
$ wget http://seanlahman.com/files/database/baseballdatabank-master_2016-03-02.zip
(the latest file as of January 2016 – zip file name may change)


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Unzip the file in your new directory. 
  
Many statistics files will unpack from the file.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Examine the following file</h4>

Batting.csv

|# |NAME |DESCRIPTION|
|0 |playerID|A unique code assigned to each player. The
playerID links the data in this file with records
in other files|
|1 yearID Year
|2 stint player's stint (order of appearances in season)
|3 teamID Team
|4 lgID League
|5 G Games
|6 AB At Bats
|7 R Runs
|8 H Hits
|9 2B Doubles
|10 3B Triples
|11 HR Homeruns
|12 RBI Runs Batted In
|13 SB Stolen Bases
|14 CS Caught Stealing
|15 BB Base on Balls
|16 SO Strikeouts
|17 IBB Intentional walks
|19 HBP Hit by pitch
|19 SH Sacrifice hits
|20 SF Sacrifice flies
|21 GIDP Grounded into double plays

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>5. Consider following fields/columns<h4>
  
• Column # 0 (Player ID)
• Column # 1 (Year)
• Column # 7 (Runs)


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>6. Put the file `Batting.csv` into HDFS under an appropriate directory, such as `batting/input`.

You may wish to rename the file in HDFS. Use `hdfs dfs –cat` to verify the put if needed.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>6. Review and understand Baseball statistics data files</h4>

Master.csv

# NAME DESCRIPTION
0 playerID Unique ID for player
1 birthYear Year player was born
2 birthMonth Month player was born
3 birthday Day player was born
4 birthCountry Country where player was born
5 birthState State where player was born
6 birthCity City where player was born
7 deathYear Year player died
8 deathMonth Month player died
9 deathDay Day player died
10 deathCountry Country where player died
11 deathState State where player died
12 deathCity City where player died
13 nameFirst Player's first name
14 nameLast Player's last name
15 nameGiven Player's given name (typically first and
middle)
16 weight Player's weight in pounds
17 height Player's height in inches
18 bats Player's batting hand (left, right, or both)
19 throws Player's throwing hand (left or right)
20 debut Date that player made first major league
appearance
21 finalGame Date that player made first major league
appearance (blank if still active)
22 more fields

For this lab, you should consider following fields/columns:

#### Batting.csv
• Column # 0 (Player ID)
• Column # 1 (Year)
• Column # 7 (Runs)

#### Master.csv
• Column # 0 (Player ID)
• Column # 13 (First Name)
• Column # 14 (Last Name)


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Identify players who scored highest runs for each year

Activity Procedure
Step 1. If you haven’t yet, create a new directory, such as baseball/input in HDFS and put statistic
files Batting.csv & Master.csv into this directory from your local machine. You may rename them in
HDFS as you wish.
Step 2. Start the Pig shell
Step 3. Load batting data into Pig using the PigStorage() function.
NOTE: The default delimiter in Pig is a TAB (\t). A CSV file has comma-separated data in each
line, so we need to inform Pig explicitly about that field delimiter. See here for the correct
function syntax.
Step 4. Read relevant fields from the loaded data. In this case we are interested in 1st , 2nd and 9th fields
for each record. Use FOREACH-GENERATE statements to accomplish this task. Look here for correct
FOREACH syntax.
HINT: In Pig Latin $0 can represent the 1st field, $1 the 2nd field and so on. In the GENERATE
statement, you can use the “$X AS (alias:type)” syntax to create text aliases for positional fields.
More on this here.
Step 5. Group runs from step 4 by year. See the syntax for GROUP here.
HINT: use DUMP and DESCRIBE to validate your assumptions along the way
Step 6. Use FOREACH-GENERATE, GROUP and MAX functions in Step 5 data to get max runs for
each year.
Step 7. Now, join Step 4 and Step 6 data based on the ‘year’ and ‘runs’ fields. Inner and outer joins
syntax is shown here.
Step 8. To identify the playerID who scored the highest one for each year, create a new bag with Year,
PlayerID and Max Run data using FOREACH, GENERATE on Step7 data.
Step 9. Check the output of the above exercise using a DUMP command.
Task 4: Determine First and Last name for the each player
Activity Procedure
Step 1. Load master data to Pig using the PigStorage() function.
Step 2. Read relevant fields from the file. In this case we are interested in 13th & 14th fields for each
record using FOREACH and generate command:
Step 3. Join PLAYERS dataset with the result dataset from previous task (Step 9) based on the common
field ‘playerID’:
Step 4. Create a new dataset having Year, Player’s First and Last Name and the MAX from Step 3 using
FOREACH and GENERATE commands.
Step 5. Make sure that data in Step 4 is sorted on Year in ascending order
Step 6. Get the output of Step 5 using DUMP command
Task 5: [OPTIONAL] Run a similar job using Pig Streaming
Activity Procedure
The syntax for Pig streaming is here.
Step 1. Write a simple script to take stdin and massage to stdout
Step 2. In Pig, LOAD data in using PigStorage() syntax.
Step 3. Use DEFINE to reference your script in Pig
Step 4. Try to come up with similar results as before.

