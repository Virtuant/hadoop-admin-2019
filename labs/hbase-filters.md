# Links for this lab: 

https://www.cloudera.com/documentation/enterprise/5-5-x/topics/admin_hbase_filtering.html

http://hbase.apache.org/0.94/book/client.filter.html

https://acadgild.com/blog/different-types-of-filters-in-hbase-shell/

https://community.hortonworks.com/questions/102061/hbase-shell-command-to-keep-filter-on-multiple-col.html

https://www.safaribooksonline.com/library/view/hbase-the-definitive/9781449314682/ch04.html


## Lab: HBase Filters

## Objective:
Learn how to use the HBase Java API to programmatically scan with a filter. 

### File locations:

### Successful outcome:
You will use the HBase Java API to programmatically scan with a Filter.


### Lab Steps

Complete the setup guide

**HBase tables**:  `movie`

In this exercise you will use the HBase Java API to programmatically scan with a Filter.

----

1.  Using an HBase scan and filters, go through every row in the movie table and find those that match the following criteria:

- The movie’s genre is “Comedy”
- The movie was made in the 1990s
- The movie’s average rating is in the 4s

Output the row key, movie name, and average rating for the rows that match these criteria.
