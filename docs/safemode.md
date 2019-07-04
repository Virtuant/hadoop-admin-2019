## Understanding Safemode in Hadoop

### Goals

Safemode is a representation that Hadoop is in read-only mode. It is ideally meant for transitioning the cluster from production to maintenance mode. In this mode, HDFS doesn’t allow any write-related and processing operations. 

However, read is possible. When we start Hadoop services, HDFS service goes in safemode to perform the following activities: 

1. Flush edits to fsimage and load fsimage in-memory
2. Get the block report from each DataNode and check the compliance of metadata
3. Report and update replication parameters (under-replicated and over-replicated blocks)

However, a user can bring the Hadoop cluster in safemode for the following reasons: 

1. Check-pointing metadata
2. Removing orphaned blocks
3. Removing corrupted blocks and its metadata.


**HDFS Files:** none
 
----

### Safe Mode

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. How to control safemode activity</h4>

To check whether the cluster is in safemode or not:

```console
 hdfs dfsadmin -safemode get 
```

To switch the cluster from production to maintenance mode: 

```console
hdfs dfsadmin -safemode enter 
```
To switch the cluster from maintenance to production mode:

```console
hdfs dfsadmin -safemode leave 

To check whether the HDFS is out of safe mode or not: 

```console
hdfs dfsadmin -safemode wait
```

### How to do Checkpointing Manually?

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Ensure the Hadoop cluster is in safemode</h4>


```console
hdfs dfsadmin -safemode enter 
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Commit the edit logs to fsimage</h4>

```console
hdfs dfsadmin -saveNamespace
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Bring the cluster back to production mode</h4>

```console
 hdfs dfsadmin -safemode leave
```

### Results

Congrats!

You see how the nodes can be checkpointed manually.

