## SPARK Catalyst at Work

### Overview
 
In this lab, we will look at several transformations and examine the optimizations that Catalyst performs.
 
----

We'll first work with the Wikimedia page count data again, and see how Catalyst helps to optimize queries involving filtering.

* Loaded the data, and then split on whitespace.
* Create a new dataframe with a finer-grained schema.
* We illustrate doing that below.

```scala
// Load the file into Spark
> val viewsDF=spark.read.text("[path to file]")
// Split on whitespace
> val splitViewsDF = viewsDF.select(split('value, "\\s+").as("splitLine"))
// Use a better schema
> val viewsWithSchemaDF = splitViewsDF.select('splitLine(0).as("domain"), 'splitLine(1).as("pageName"), 'splitLine(2).cast("integer").as("viewCount"), 'splitLine(3).cast("long").as("size"))
```

* If you haven't already done the steps above, then do so now.
	* Cat/copy from the helper file, __*spark-labs/helpers/wikiSchema.txt*__, and use the `:paste` command in the shell.

### Push Down Predicate

* First, write a transformation to order `viewsWithSchemaDF` by `viewCount`.
* `explain` the above transformation.
	* Note that there is shuffling involved (an `Exchange`).
	* It needs to shuffle data to sort the rows.

* Next, filter `viewsWithSchemaDF` so you only view rows where the domain starts with "en".
	* Put the filter at the end of the transformation chain in your code (after the ordering).
	* You can use `startsWith("en")` on the domain column to accomplish this.
	* `explain` this transformation.
	* You should see the filtering happening before the shuffle for ordering.
	* Catalyst has **pushed the filter down** to improve efficiency.
	* `explain(true)` the transformation to see the steps that Catalyst took.
* Optionally, try the transformation with the filter placed before the ordering.
	* It should give you exactly the same plan.

### Work with DataSets and lambdas

We'll now create a DataSet and filter it using a lambda.  We'll look at how the lambda affects the Catlyst optimizations.

* Declare a case class for the Wikimedia data.
* Create a DataSet using the case class.
* We show this code below, and also supply it in __*spark-labs/helpers/wikiSchema.txt*__ for easy copy/paste

```scala
case class WikiViews(domain:String, pageName:String, viewCount:Integer, size:Long)
val viewsDS = viewsWithSchemaDF.as[WikiViews]
```

* Next, filter `viewsDS` so you only view rows where the domain starts with "en".
	* Put the filter at the end of the transformation chain (after the ordering)
	* You have to use a lambda for this.
	* You can use `startsWith("en")` on the domain value to accomplish this (this is a Scala function on strings).
	* `explain` this transformation.
* Where does the filtering happen?  It should be the very last thing.  Why?

### Review Other Transformations

* If you have time, `explain` any of the other transformations you've been working with to see how Catalyst might have optimized them.
* If you have `folksDF` (the dataframe with people data in it) still active in your Spark shell, then explain the following transformation, and note how the filter is pushed down into the file scan - about as efficient as you can get.
	* Look at the **PushedFilters:** entry

```scala
> folksDF.filter('age>25).orderBy('age).explain
== Physical Plan ==
*Sort [age#42L ASC NULLS FIRST], true, 0
+- Exchange rangepartitioning(age#42L ASC NULLS FIRST, 200)
   +- *Project [age#42L, gender#43, name#44]
      +- *Filter (isnotnull(age#42L) && (age#42L > 25))
         +- *FileScan json [age#42L,gender#43,name#44] Batched: false, Format: JSON, Location: ..., PartitionFilters: [], PushedFilters: [IsNotNull(age), GreaterThan(age,25)], ReadSchema: struct<age:bigint,gender:string,name:string>
```

### Results

**Catalyst rules!**

We've explained it, now you see it.

But lambdas can overthrow the ruler - so be cautious with them.
