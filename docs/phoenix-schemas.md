## Schemas on HBase with Phoenix

**Objective**: Use Phoenix to manipulate schemas.

In this exercise you will use the Phoenix to get around in HBase schemas.

----

Look at your tables again.

```sql
0: jdbc:phoenix:localhost> !tables
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
| TABLE_CAT  | TABLE_SCHEM  |       TABLE_NAME        |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENC |
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
|            | SYSTEM       | CATALOG                 | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | FUNCTION                | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | LOG                     | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | SEQUENCE                | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | STATS                   | SYSTEM TABLE  |          |            |               |
|            |              | DRIVER                  | TABLE         |          |            |               |
|            |              | driver_dangerous_event  | TABLE         |          |            |               |
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
0: jdbc:phoenix:localhost> 
```

These tables other than SYSTEM tables, have all been put into the default schema. What about schemas?

OK, glad you asked. Start by dropping the table created before:

```sql
0: jdbc:phoenix:localhost> drop TABLE DRIVER;
No rows affected (1.573 seconds)
```

Now we'll recreate the table with a schema:

```sql
0: jdbc:phoenix:localhost> create table s1.driver (id integer primary key);
No rows affected (0.742 seconds)
```

And check it:

```sql
0: jdbc:phoenix:localhost> !tables
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
| TABLE_CAT  | TABLE_SCHEM  |       TABLE_NAME        |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENC |
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
|            | SYSTEM       | CATALOG                 | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | FUNCTION                | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | LOG                     | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | SEQUENCE                | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | STATS                   | SYSTEM TABLE  |          |            |               |
|            |              | driver_dangerous_event  | TABLE         |          |            |               |
|            | S1           | DRIVER                  | TABLE         |          |            |               |
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
0: jdbc:phoenix:localhost> 
```

Now create another schema:

```sql
0: jdbc:phoenix:localhost> create table s2.driver (id integer primary key);
No rows affected (0.736 seconds)
```

Now check again:

```sql
0: jdbc:phoenix:localhost> !tables
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
| TABLE_CAT  | TABLE_SCHEM  |       TABLE_NAME        |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENC |
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
|            | SYSTEM       | CATALOG                 | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | FUNCTION                | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | LOG                     | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | SEQUENCE                | SYSTEM TABLE  |          |            |               |
|            | SYSTEM       | STATS                   | SYSTEM TABLE  |          |            |               |
|            |              | driver_dangerous_event  | TABLE         |          |            |               |
|            | S1           | DRIVER                  | TABLE         |          |            |               |
|            | S2           | DRIVER                  | TABLE         |          |            |               |
+------------+--------------+-------------------------+---------------+----------+------------+---------------+
0: jdbc:phoenix:localhost> 
```

Now let us add a column to our new table:

```sql
0: jdbc:phoenix:localhost> alter table s2.driver add fname varchar;
No rows affected (5.773 seconds)
```

Select the table:

```sql
0: jdbc:phoenix:localhost> select * from s2.driver;
+-----+--------+
| ID  | FNAME  |
+-----+--------+
+-----+--------+
No rows selected (0.012 seconds)
```

You can use the shorhand command to see how the table is structured, for both schemas:

```sql
0: jdbc:phoenix:localhost> !columns driver;
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LE |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
|            | S1           | DRIVER      | ID           | 4          | INTEGER    | null         | null      |
|            | S2           | DRIVER      | ID           | 4          | INTEGER    | null         | null      |
|            | S2           | DRIVER      | FNAME        | 12         | VARCHAR    | null         | null      |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
0: jdbc:phoenix:localhost> 
```

Or be specific as to schema:


```sql
0: jdbc:phoenix:localhost> !columns s2.driver;
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LE |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
|            | S2           | DRIVER      | ID           | 4          | INTEGER    | null         | null      |
|            | S2           | DRIVER      | FNAME        | 12         | VARCHAR    | null         | null      |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
0: jdbc:phoenix:localhost> 
```

And if we add a column:


```sql
0: jdbc:phoenix:localhost> alter table s2.driver add lname varchar;
No rows affected (6.142 seconds)
```

Now the s2 table becomes:

```sql
0: jdbc:phoenix:localhost> !columns s2.driver;
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LE |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
|            | S2           | DRIVER      | ID           | 4          | INTEGER    | null         | null      |
|            | S2           | DRIVER      | FNAME        | 12         | VARCHAR    | null         | null      |
|            | S2           | DRIVER      | LNAME        | 12         | VARCHAR    | null         | null      |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
0: jdbc:phoenix:localhost> 
```

Now lets add 1 more column:

```sql
0: jdbc:phoenix:localhost> alter table s2.driver add eqwfwef varchar;
No rows affected (5.761 seconds)
```

And now we have in s2:

```sql
0: jdbc:phoenix:localhost> !columns s2.driver;
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LE |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
|            | S2           | DRIVER      | ID           | 4          | INTEGER    | null         | null      |
|            | S2           | DRIVER      | FNAME        | 12         | VARCHAR    | null         | null      |
|            | S2           | DRIVER      | LNAME        | 12         | VARCHAR    | null         | null      |
|            | S2           | DRIVER      | EQWFWEF      | 12         | VARCHAR    | null         | null      |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
0: jdbc:phoenix:localhost> 
```

Wait a minute! We made a mistake. So what to do now?

```sql
0: jdbc:phoenix:localhost> alter table s2.driver drop column eqwfwef;
No rows affected (0.024 seconds)
```

And now the schema/table shows:

```sql
0: jdbc:phoenix:localhost> !columns s2.driver;
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  | COLUMN_NAME  | DATA_TYPE  | TYPE_NAME  | COLUMN_SIZE  | BUFFER_LE |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
|            | S2           | DRIVER      | ID           | 4          | INTEGER    | null         | null      |
|            | S2           | DRIVER      | FNAME        | 12         | VARCHAR    | null         | null      |
|            | S2           | DRIVER      | LNAME        | 12         | VARCHAR    | null         | null      |
+------------+--------------+-------------+--------------+------------+------------+--------------+-----------+
0: jdbc:phoenix:localhost> 
```

### Results

So now you see how schemas can be used to appropritely separate the data in rows from one another. And you can treat the either together or separate as your needs require.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>