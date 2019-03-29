## Analyzing Big Data with Hive

**Objective:** Analyze the White House visitor data. You will have discovered several useful pieces of
information about the White House visitor data.

**Data Set** [Constitution.txt] (https://mapr.com/products/mapr-sandbox-hadoop/tutorials/hue-tutorial-file-browser-metastore-manager-beeswax/assets/constitution.txt)

----

### Find the First Visit

Using touch, create a new text file named `whitehouse.hive` and save it.

In this step, you will instruct the hive script to find the first visitor to the
White House (based on our dataset). This will involve some clever
handling of timestamps. This will be a long query, so enter it on multiple
lines (note the lack of a ";" at the end of this first step). 

Start by selecting
all columns where the time_of_arrival is not empty:

```sql
select * from wh_visits where time_of_arrival != ""
```

To find the first visit, we need to sort the result. This requires converting
the time_of_arrival string into a timestamp. We will use the
unix_timestamp function to accomplish this. Add the following order by
clause (again, no ";" at the end of the line):
order by unix_timestamp(time_of_arrival,
'MM/dd/yyyy hh:mm')
e. Since we are only looking for one result, we certainly don’t need to
return every row. Let’s limit the result to 10 rows, so we can view the first
10 visitors (this finishes the query, so will end with the ";" character):
limit 10;
f. Save your changes to whitehouse.hive.

Execute the script whitehouse.hive and wait for the results to be
displayed:
# cd ~/devph/labs/Lab7.2
# hive -f whitehouse.hive
h. The results should be 10 visitors, and the first visit should be in 2009,
since that is when the dataset begins. The first visitors are Charles Kahn
and Carol Keehan on 3/5/2009.
2 ) Find the Last Visit
a. This one is easy: just take the previous query and reverse the order by
adding desc to the order by clause:
order by unix_timestamp(time_of_arrival,
'MM/dd/yyyy hh:mm') desc
b. Run the query again, and you should see that the most recent visit was
Jackie Walker on 3/18/2011.
# hive -f whitehouse.hive
3 ) Find the Most Common Comment
a. In this step, you will explore the info_comment field and try to determine
the most common comment. You will use some of Hive’s aggregate
functions to accomplish this. Start by using gedit to create a new text file
named comments.hive and save it in ~/devph/labs/Lab7.2 folder.
b. You will now create a query that displays the 10 most frequently
occurring comments. Start with the following select clause:
from wh_visits
select count(*) as comment_count, info_comment
This runs the aggregate count function on each group (which you will define later in
the query) and names the result comment_count. For example, if “OPEN HOUSE” occurs
five times then comment_count will be five for that group.
Notice we are also selecting the info_comment column so we can see what the
comment is.
c. Group the results of the query by the info_comment column:
group by info_comment

Order the results by comment_count, because we are only interested in
comments that appear most frequently:
order by comment_count DESC
e. We only want the top results, so limit the result set to 10:
limit 10;
f. Save your changes to comments.hive and execute the script. Wait for the
MapReduce job to execute.
# hive -f comments.hive
g. The output will be 10 comments and should look like:
9036
1253 HOLIDAY BALL ATTENDEES/
894 WHO EOP RECEP 2
700 WHO EOP 1 RECEPTION/
601 RESIDENCE STAFF HOLIDAY RECEPTION/
586 PRESS RECEPTION ONE (1)/
580 GENERAL RECEPTION 1
540 HANUKKAH RECEPTION./
540 GEN RECEP 5/
516 GENERAL RECEPTION 3
h. It appears that a blank comment is the most frequent comment, followed
by the HOLIDAY BALL, then a variation of other receptions.
i. Modify the query so that it ignores empty comments. If it works, the
comment “GEN RECEP 6/” will show up in your output.
Solution:
--In comments.hive, insert the following line between your select
and group statements:
where info_comment != ""
Save the changes, then back at the command line, re-run the query:
# hive -f comments.hive


Least Frequent Comment
a. Run the previous query again, but this time, find the 10 least occurring
comments.
--Remove DESC from your order statement so that it looks like
this:
order by comment_count
Save the changes, then back at the command line, re-run the query:
# hive -f comments.hive
The output should look something like:
1 CONGRESSIONAL BALL/
1 CONG BALL/
1 merged to u59031
1 CONGRESSIONAL BALL
1 CONG BALL
1 COMMUNITY COLLEGE SUMMIT
1 48 HOUR WAVE EXCEPTION GRANTED
1 DROP BY VISIT
1 WHO EOP/
1 "POTUS LUNCH WITH WASHINGTON
This seems accurate since 1 is the least number of times a comment can
appear.
5 ) Analyze the Data Inconsistencies
a. Analyzing the results of the most- and least-frequent comments, it
appears that several variations of GENERAL RECEPTION occur. In this step,
you will try to determine the number of visits to the POTUS involving a
general reception by trying to clean up some of these inconsistencies in
the data.
Note
Inconsistencies like these are very common in big data, especially when
human input is involved. In this dataset, we likely have different people
entering similar comments but using their own abbreviations.
b. Modify the query in comments.hive. Instead of searching for empty
comments. Search for comments that contain variations of the string
“GEN RECEP.”
where info_comment rlike '.*GEN.*\\s+RECEP.*'

Change the limit clause from 10 to 30:
limit 30;
d. Run the query again.
# hive -f comments.hive
e. Notice there are several GENERAL RECEPTION entries that only differ by a
number at the end or use the GEN RECEP abbreviation:
580 GENERAL RECEPTION 1
540 GEN RECEP 5/
516 GENERAL RECEPTION 3
498 GEN RECEP 6/
438 GEN RECEP 4
31 GENERAL RECEPTION 2
23 GENERAL RECEPTION 3
20 GENERAL RECEPTION 6
20 GENERAL RECEPTION 5
13 GENERAL RECEPTION 1
f. Let’s try one more query to try and narrow GENERAL RECEPTION visit.
Modify the WHERE clause in comments.hive to include “%GEN%”:
where info_comment like "%RECEP%"
and info_comment like "%GEN%"
g. Leave the limit at 30, save your changes, and run the query again.
# hive -f comments.hive
h. The output this time reveals all the variations of GEN and RECEP. Next, let’s
add up the total number of them by running the following query:
from wh_visits
select count(*)
where info_comment like "%RECEP%"
and info_comment like "%GEN%";
--Then save your changes and run the query again from the command
line:
# hive -f comments.hive
i. Notice there are 2,697 visits to the POTUS with GEN RECEP in the
comment field, which is about 12% of the 21,819 total visits to the
POTUS in our dataset.

Note
More importantly, these results show that the conclusion from our first
query, where we found that the most likely reason to visit the President
was the HOLIDAY BALL with 1,253 attendees, is incorrect. This type of
analysis is common in big data, and it shows how data analysts need to
be creative and thorough when researching their data.
6 ) Verify the Result
a. We have 12% of visitors to the POTUS going for a general reception, but
there were a lot of statements in the comments that contained WHO and
EOP. Modify the query from the last step and display the top 30
comments that contain “WHO” and “EOP.”
--You should be able to undo changes to comments.hive and restore
it to the state before the last lab. Then make the following two
additional edits:
--Change the where clause to match WHO and EOP
where info_comment like "%WHO%"
and info_comment like "%EOP%";
--Add the DESC command back to the end of the order statement
order by comment_count DESC
--Finally, double-check select count(*) as comment_count,
info_count
--Make sure the "as..." portion is there
--Then save your changes and run the query again from the command
line:
# hive -f comments.hive

The result should look like:
894 WHO EOP RECEP 2
700 WHO EOP 1 RECEPTION/
43 WHO EOP RECEP/
20 WHO EOP HOLIDAY RECEP/
13 WHO/EOP #2/
8 WHO EOP RECEPTION
7 WHO EOP RECEP
1 WHO EOP/
1 WHO EOP RECLEAR
b. Modify the script again, this time to run a query that counts the number
of records with WHO and EOP in the comments, and run the query:
from wh_visits
select count(*)
where info_comment like "%WHO%"
and info_comment like "%EOP%";
--Run the query from the command line:
# hive -f comments.hive
You should get 1,687 visits, or 7.7% of the visitors to the POTUS. So
GENERAL RECEPTION still appears to be the most frequent comment.
7 ) Find the Most Visits
a. See if you can write a Hive script that finds the top 20 individuals who
visited the POTUS most. Use the Hive command from Step 3 earlier in
this lab as a guide.
Tip
Use a grouping by both fname and lname.
The following script will accomplish the intention of the previous step:
from wh_visits
select count(*) as most_visit, fname, lname
group by fname, lname
order by most_visit DESC
limit 20;

To verify that your script worked, here are the top 20 individuals who
visited the POTUS along with the number of visits (your output may vary
slightly due to randomization of names):
16 ALAN PRATHER
15 CHRISTOPHER FRANKE
15 ANNAMARIA MOTTOLA
14 ROBERT BOGUSLAW
14 CHARLES POWERS
12 SARAH HART
12 JACKIE WALKER
12 JASON FETTIG
12 SHENGTSUNG WANG
12 FERN SATO
12 DIANA FISH
11 JANET BAILEY
11 PETER WILSON
11 GLENN DEWEY
11 MARCIO BOTELHO
11 DONNA WILLINGHAM
10 DAVID AXELROD
10 CLAUDIA CHUDACOFF
10 VALERIE JARRETT
10 MICHAEL COLBURN
Result
You have written several Hive queries to analyze the White House visitor data. The
goal is for you to become comfortable with working with Hive, so hopefully you now
feel like you can tackle a Hive problem and be able to answer questions about your
big data stored in Hive.

