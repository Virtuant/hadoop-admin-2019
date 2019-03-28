## Exploring Grouping

**Overview** In this lab, we will work with the Github archive that contains activity for a single day.  We'll analyze it to find the top contributors for that day.

***Data Set** `data/github.json` that contains a log of all github activity for one day.  

### Github Activity Archive

* Loaded the data, and viewed the schema
* Selected the actor column, which has a nested structure, and worked with some of its subelements
* We illustrate doing that below

```scala
// Load the file into Spark
> val githubDF=spark.read.json("data/github.json")
// Select the actor column
> val actorDF = githubDF.select('actor)
// Print actor schema
> actorDF.printSchema
// Select the actor's login value - note how we 
// Use a SQL expression in the select, not a Column
> actorDF.select("actor.login").limit(5).show

```

### Tasks

* You'll only reuse the githubDF dataframe in this lab
* The other statements are to practice working with the schema, which is complex

### Query the Data by Actor's Login Value

* Query the github data for how many entries exist in the data for each actor's login.  Use the DSL
* **Hints**: 
  * You'll want to group the data by the actor's login
	* You'll probably want to use an SQL expression to express the actor's login, not a column value
	* You want a count of entries for each login value
* Show a few rows of this data
* Lastly, find the 20 logins with the largest number of contributions, and display them

### Use SQL

* Optionally, try doing the above query in SQL.
* It's pretty much standard SQL, so if you know that well, it's not very complex.
* Remember to create a temporary view (`createOrReplaceTempView`)

### Summary

The task we did in this lab is not overly complex, but it's also not trivial.  Spark SQL makes it reasonable for us to accomplish this in a short lab, either using the DSL, or using SQL.

If you wanted to do this using RDDs, it would be a much more complex series of transformations - starting with the messy ones of parsing the JSON data.  This is why Spark SQL is becoming the standard API for working with Spark.

