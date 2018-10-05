## Lab: HBase Filters

**HBase tables**:  `movie`

In this exercise you will use the HBase Java API to programmatically scan with a Filter.

----

1.  Using an HBase scan and filters, go through every row in the movie table and find those that match the following criteria:

- The movie’s genre is “Comedy”
- The movie was made in the 1990s
- The movie’s average rating is in the 4s

Output the row key, movie name, and average rating for the rows that match these criteria.
