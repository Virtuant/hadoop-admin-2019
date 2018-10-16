## Analysing MovieLens movie data using Pig


This dataset has been collected by GroupLens Research Project. Three datasets of varying sizes are available 
from the site. These are MovieLens 100K, MovieLens 1M and MovieLens 10M. The datasets contain movie ratings made by 
movie goers. We will use MovieLens 1M dataset. Download and unzip it. It contains three text files: ratings.dat, 
users.dat and movies.dat. You can view them in gedit if you have sufficient RAM. 
Contents of these files are described in README that accompanies the dataset. 
For the sake of completeness, data in the three files is briefly described here.

**Lab files**: We will use MovieLens dataset for analysis with Pig. If needed, the data is available from 
[here](https://grouplens.org/datasets/movielens).

----

### The Data

```
ratings.dat–>userid::movieid:rating::timestamp
```

* userid ranges from 1 to 6040
* movieid ranges from 1 to 3952
* rating is made on a five-star discrete scale (whole-star rating only)
* timestamp is in seconds.

```
users.dat–>userid::gender::age::occupation::zipcode
```

* gender is M or F
* age is categorized as 1, 18, 25, 35,45, 50, 56
* The meanings are: 1: <18; 18: 18-24; 25: 25-34; 35: 35-44; 45: 45-49; 50: 50-55; 56 is 56+
* occupation is one of these: 
	0: other
	1: academic/educator
	2: artist
	3: clerical/admin
	4: college/grad student
	5: customer service
	6: doctor/health care
	7: executive/managerial
	8: farmer
	9: homemaker
	10: K-12 student
	11: lawyer
	12: programmer
	13: retired
	14: sales/marketing
	15: scientist
	16: self-employed
	17: technician/engineer
	18: tradesman/craftsman
	19: unemployed
	20: writer

```
movies.dat–>movieID::title::genres
```

Titles are movie titles and genres are pipe-separated and are selected from the following genres: 

* Action, Adventure, Animation, Children’s, Comedy, Crime, Documentary, Drama, Fantasy, Film-Noir, Horror, Musical, Mystery, Romance, Sci-Fi, Thriller, War, Western

Field separators are double colon. We substitute them with semicolon. And then copy the whole directory to hadoop folder. The following shell script does the preparatory work of inserting semicolon as field separator and copying the whole movie folder to hadoop.

So create this script in the system:

```bash
#!/bin/bash
#
#
unzip ml-1m.zip 
mv ml-1m movie
# Unfortunately PigStorage does not work with :: as separators
# so replace :: with ;
sed 's/\:\:/;/g' users.dat > users.txt
sed 's/\:\:/;/g' movies.dat > movies.txt
sed 's/\:\:/;/g' ratings.dat > ratings.txt
mv users.txt users.dat
mv movies.txt movies.dat
mv ratings.txt ratings.dat
```

### Copy files to Hadoop system

# Delete any earlier movie folder on hadoop

```
hdfs dfs -rmr movie
```

# Copy movie folder to hdfs

```
hdfs dfs -put movie
```

Pig can be started from logged in user’s command shell as below. If you are on Cloudera system, you may have to export Classpath of hadoop libraries. After a few messages (grunts), pig shell prompt ‘grunt’ appears. We will execute pig commands in this shell.

```
$ pig
grunt> 
```

Using load statement, load user.dat file into pig. In fact, nothing is loaded immediately. Only syntax is checked. Whether file exists is also not found out at this stage.

Load users.dat file. 'user' on the left is not a variable but an alias for the load command. 
While loading, we also: <strong>a. </strong>indicate the field separator, <strong>b.</strong> specify proposed field names, and <strong>c.</strong> specify their data-types. The default field separator is tab.
 
```
user = load '/user/ashokharnal/movie/users.dat' using PigStorage(';') as 
	(userid:int, gender:chararray, age:int, occupation:int, zip:chararray) ;
```

For every row in the user, generate columnar data as follows. This statement, in effect, projects named columns

```
udata = foreach user generate userid , gender , age ,occupation , zip ;
```
 
Dump output to screen but limit it to five rows

```
lt = limit udata 5 ;
dump lt ;
```

In pig, function names and alias are case sensitive. The pig latin statements are not case sensitive.
While executing load, foreach and limit statements, pig will merely check the syntax. Only when dump (or store) statement is encountered, these statements are fully executed and output produced.

Let us now get the distribution of gender in the data. We will first group by gender using group statement and then run a foreach statement (followed by dump) to see some columns. (Note: In pig a comment can be written in c-style or after two dashes (–) on a line.

>Note: you don't need to enter comments

```
gr = group user by gender ;                      --group alias 'user' by gender
co = foreach gr generate group, COUNT(user) ;    --for each line in the group (i.e. M and F)
                                                 -- count rows in 'user'
co = foreach gr generate group, COUNT_STAR(user) ; --same as above
dump co ;
```

Let us now count how many persons are below 34 years of age. We want to get the number of young users.
	
First write a filter statement

```
young = filter user by age < 35 ; 
cm = group young all ;        
countyoung = foreach cm generate COUNT(young) ;
dump countyoung ;
```

In the above code, you may find the occurrence of ‘group‘ statement a little unusual. The group statement groups all fields (and hence is practically of no use). FOREACH statement will not work on a filter alias i.e. young (i.e. foreach young…). Count next how many females users between age category 35 and 50 have rated movies. We will use two filters, one after another.


How many are between 35 and 59?

```
m_age = filter user by age >=35 AND age <= 50 ;
```

How many of these are females. Filter after filter?

```
m_age_female = filter m_age by gender == 'F' ; 
```

First create group to project a column

```
cmg = group m_age_female by gender ;  -- group by gender, OR
cmg = group m_age_female ALL ;        -- group by ALL fields
```

Note that above group statement is NOT 'group m_age_female <em>by</em> ALL. Create a foreach statement and dump it:

```
count_m_female = foreach cm generate COUNT(m_age_female) ;  
dump count_m_female ;
```

We can order ‘age’ wise our filtered data using order statement:

```
ord = order m_age_female by age ;
```

NExt, we split data in various ways. We will first split it gender wise and then age wise.

Split udata by gender into two aliases:

```
split udata into female if gender == 'F', male if gender == 'M' ;
dump female ;   --see female data only
```

Split data into young and middle aged females:

```
split udata into yfemale if gender == 'F' AND age <= 35, mfemale if ( gender == 'F' AND (age > 35 AND age <= 50 ));
dump yfemale;   -- see young females data
```

Sample 10% of users data for experimentation. The sample is random but no other data file is created that excludes records contained in the sample. Sample 10% (0.01) of data:

```
sam = sample udata 0.01 ;
dump sam;
```

Let us now load `ratings` data and generate columns.

Load ratings file in pig. Also specify column separator, column names and their data types:

```
ratings = load '/user/ashokharnal/movie/ratings.dat' using PigStorage(';') 
	as (userid:int, movieid:int, ratings:int, timestamp:int) ;
```

A foreach statement to generate columns. You can dump alias, rdata:

```
rdata = foreach ratings generate (userid,movieid,ratings,timestamp) ;  --dump will have extra brackets (( ))
rdata = foreach ratings generate (userid,movieid,ratings,timestamp) ;  --dump will have one pair of brackets
```

Now check:

```
x = limit rdata 5 ;
dump x ;
```

We will now join the two relations ‘user’ and ‘ratings’ on userid. This is an inner join. We will use the join to calculate average ratings given by females and males. Are females more generous?
	
--Join two relations on userid.
j = join user by userid, ratings by userid ;
/* Group join by gender */
h = group j by gender;
/* Write a foreach to work group wise.
'group' stands for group-field with two values (M & F)
Note also that ratings field within AVG needs to be qualified by alias */
g = foreach h generate group, AVG(j.ratings) ;
/* The following will also produce output but massive repetition of M,M..*/
g = foreach h generate j.gender , AVG(j.ratings) ;

The result is: (F,3.6203660120110372) (M,3.5688785290984373). You can draw your own conclusions. Next let us load movies.dat file and dump a few lines for our inspection.
1
2
3
4
5
	
-- Load movies.dat 
movie = load '/user/ashokharnal/movie/movies.dat' using PigStorage(';') as (movieid:int, title:chararray, genres:chararray) ;
movielist = foreach movie generate movieid, title, genres ;
x = limit movielist 3 ;
dump x;

We will now join all the three tables: user, ratings and movies. We will join ‘movie’ with j, the earlier join of two relations, j. As this join is complicated, we can describe it to see how does it appear.
1
2
3
4
5
6
7
8
9
10
11
	
# Join the three tables
jo = join  j by movieid, movie by movieid ;
describe jo ;
jo: {movie::movieid: int,movie::title: chararray,movie::genres: chararray,j::user::userid: int,j::user::gender: chararray,j::user::age: int,j::user::occupation: int,j::user::zip: chararray,j::ratings::userid: int,j::ratings::movieid: int,j::ratings::ratings: int,j::ratings::timestamp: int}
 
y = limit jo 3 ;
dump jo ;
/* Project few fields of triple join. Note how field-names are accessed.
ghost = foreach jo generate j::user::userid  , j::ratings::ratings as rate , movie::genres ;
tt = limit ghost 5 ;
dump tt ;

Let us create a filter on this triple join. We are interested only in ‘Comedy’ movies. Only if the word ‘Comedy’ appears in genre string, it is of relevance to us.
1
2
3
4
	
-- Filter out only Comedy from genres. Note carefully the syntax
filt = filter ghost by ( movie::genres matches '.*Comedy.*' ) ;
relx = group filt by movie::genres ;
dump relx ;

As field-naming in triple join becomes a little complex, it is better to store the result in hadoop and reload the stored file in pig for further manipulation. We will do this now.
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
	
/* Store jo and reload it. This simplifies query. For storing,
in hadoop, only folder name is to be given. Pig will create the folder. */
 
store jo into '/user/ashokharnal/temp/myfolder' ; 
 
-- Reload the stored file within 'myfolder'. As userid appears twice, name it userid1 and userid2.
-- Do the same for other fields
 
l_again = load '/user/ashokharnal/temp/myfolder/part-r-00000' as (userid1:int, gender:chararray, age:int, occupation:int, zip:chararray, userid2:int, movieid1:int, ratings:int, timestamp:int, movieid2:int, title:chararray, genres:chararray) ;
 
--Check results
z = limit l_again 10 ;
dump z ;
 
--Create a projection of few columns for our manipulation. We will use alias 'alltable'
alltable = foreach l_again generate gender, genres, ratings ;

Let us now see whether there is any preference for comedy movies across gender.
1
2
3
4
5
6
7
8
9
10
	
--First filter out Comedy movies from triple join.
filt = filter alltable by ( genres matches '.*Comedy.*' )  ;
 
-- Group by gender 
mygr = group filt by gender ;
 
-- Find how gender votes for comedy movies
-- Note AVG argument should be as: filt.ratings. Just ratings raises an error.
vote = foreach mygr generate group, AVG(filt.ratings) ;
dump vote ;

The result is: (F,3.5719375512875113) (M,3.503666796000138)
Well both are about equal.

Happy pigging…
