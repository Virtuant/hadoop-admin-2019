# Links for this lab: 

https://www.cloudera.com/documentation/enterprise/5-5-x/topics/admin_hbase_filtering.html

http://hbase.apache.org/0.94/book/client.filter.html

https://community.hortonworks.com/questions/102061/hbase-shell-command-to-keep-filter-on-multiple-col.html

https://www.safaribooksonline.com/library/view/hbase-the-definitive/9781449314682/ch04.html

## Lab: HBase Filters

**Objective**:  Learn how to use the HBase Java API to programmatically scan with a filter. 

**File locations**:

**Successful outcome**: You will use the HBase Java API to programmatically scan with a Filter.

**HBase tables**:  `movie`

In this exercise you will use the HBase Shell to apply different filter technologies to HBase.

Filters and Filter Language permit you to perform server-side filtering when accessing HBase over Thrift or within the HBase shell.

When reading information from HBase using Get or Scan operations, you’ll be able to use custom filters to return a set of results to the client. 

This, however, doesn’t reduce server-side IO, it will only cut back network information measure and reduces the amount of information the client has to process. Filters are typically implemented using the Java API, however, are often used from HBase shell for testing and debugging purposes.

----


Ideal scan for a table named “iot-in”.



We will perform few filter operations on the table below.



the command for list filters are available in HBase

### FirstKeyOnlyFilter

This filter doesn’t take any arguments. It returns solely the primary key-value from every row.

Syntax
FirstKeyOnlyFilter ()

Example of firstkeyonlyfilter


### KeyOnlyFilter

This filter doesn’t take any arguments. It returns solely the key part of every key-value.

Syntax
KeyOnlyFilter ()

Example of keyonlyfilter


### Prefixfilter:

This filter takes one argument as a prefix of a row key. It returns solely those key-values present in the very row that starts with the specified row prefix

Syntax
PrefixFilter (<row_prefix>)

Example of prefixfilter


### ColumnPrefixFilter

This filter takes one argument as column prefix. It returns solely those key-values present in the very column that starts with the specified column prefix. The column prefix should be the form qualifier

Syntax
ColumnPrefixFilter(<column_prefix>)

Example of columnprefixfilter


### MultipleColumnPrefixFilter

This filter takes a listing of column prefixes. It returns key-values that are present in the very column that starts with any of the specified column prefixes. every column prefixes should be a form qualifier.

Syntax

MultipleColumnPrefixFilter(â€˜<column_prefix>,<column_prefix>,….<column_prefix>)

Example of multiplecolumnprefixfilter

### ColumnCountGetFilter

This filter takes one argument a limit. It returns the primary limit number of columns within the table.

Syntax

ColumnCountGetFilter(<limit>)

Example of columncountgetfilter


### Hadoop

### PageFilter

This filter takes one argument a page size. It returns page size number of the rows from the table

Syntax
PageFilter (<page_size>)l

Example of pagefilter


### InclusiveStopFilter

This filter takes one argument as row key on that to prevent scanning. It returns all key-values present in rows together with the specified row.

Syntax
InclusiveStopFilter(<stop_row_key>)

Example of Inclusivestopfilter


### Qualifier Filter (Family Filter)

This filter takes a compare operator and a comparator. It compares every qualifier name with the comparator using the compare operator and if the comparison returns true, it returns all the key-values in this column.

Syntax
QualifierFilter (<compareOp>, <qualifier_comparator>)

Example of Qualifier Filter


### ValueFilter

This filter takes a compare operator and a comparator. It compares every value with the comparator using the compare operator and if the comparison returns true, it returns that key-value.

Syntax
ValueFilter (<compareOp>,‘<value_comparator>’)

The above all filters are very basic filters in HBase shell. Let’s look at the little complex one.

### SingleColumnValueFilter

This filter as an argument takes a column family, a qualifier, a compare operator and a comparator. So, if the specified column isn’t found, all the columns of that row are going to be emitted. And ,If the column is found and also the comparison with the comparator returns true, all the columns of the row are going to be emitted. If the condition fails, the row won’t be emitted.

This filter additionally takes 2 extra optional boolean arguments – filterIfColumnMissing and setLatestVersionOnly
If the filterIfColumnMissing flag is set to true, the columns of the row won’t be emitted if the specified column to examine isn’t found within the row. The default value is false.

If the setLatestVersionOnly flag is set to false, it’ll check previous versions (timestamps) too. The default value is true.
These flags are not mandatory and if you must set neither or both.

Syntax

SingleColumnValueFilter(‘<family>’,‘<qualifier>’, <compare operator>, ‘<comparator>’, <filterIfColumnMissing_boolean>, <latest_version_boolean>)

SingleColumnValueFilter(‘<family>’, ‘<qualifier>, <compare operator>, ‘<comparator>’)


### Summery

There are more:
while, You can see the list of filters in HBase by using HBase command (show_filters)
