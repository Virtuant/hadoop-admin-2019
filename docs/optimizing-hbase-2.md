## Lab: More on Optimization of HBase


----

Now lets create another table:

```sql
hbase(main):002:0> create 'user','cf1', SPLITS => ['g','m']
Created table user
Took 2.2917 seconds
=> Hbase::Table - user
```

Now let's put in some rows:

```sql
hbase(main):007:0> put 'user','a','cf1:name','test'
hbase(main):008:0> put 'user','k','cf1:name','test'
hbase(main):009:0> put 'user','r','cf1:name','test'
```

And the scan shows:

```sql
hbase(main):010:0> scan 'user'
ROW     COLUMN+CELL
 a      column=cf1:name, timestamp=1535736598392, value=test
 k      column=cf1:name, timestamp=1535736606360, value=test
 r      column=cf1:name, timestamp=1535736615460, value=test
3 row(s)
Took 0.0210 seconds
```

Now go to the HBase Master UI:

![image](https://user-images.githubusercontent.com/558905/44928794-f8d6f680-ad26-11e8-9d4d-039960e10e42.png)

See how your regions are split?

Now increment the reads:

```sql
hbase(main):024:0> get 'user','a'
COLUMN              CELL
 cf1:name           timestamp=1535736598392, value=test
1 row(s)
Took 0.0134 seconds
hbase(main):025:0> get 'user','r'
COLUMN              CELL
 cf1:name           timestamp=1535736615460, value=test
1 row(s)
Took 0.0042 seconds
```

And check the Metrics tab:

![image](https://user-images.githubusercontent.com/558905/44929318-ab5b8900-ad28-11e8-838a-a3d44f279f0b.png)

Now you see how HBase can be tuned by these region splits.

