## Decommisoning and Whitelisting HDFS Nodes in an Existing Cluster

### Goals

Decommissioning is a process by which we gracefully remove the DataNode from the running cluster without affecting any storage and processing activities triggered by Hadoop or similar applications. 

The responsibility of that specific DataNode which is planned to be decommissioned will be assigned to other DataNodes and the NameNode will keep a track of it and update the same in the metadata.

**HDFS Files:** `hdfs-site.xml`
 
----

### Decommissioning Datanode in Existing Hadoop Cluster

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Create a file in the NameNode system</h4>

In `node1` that holds the hostname of the node which is to be decommissioned, you can set any name to this file. 

In this case, let us create a file named ‘remove.txt’:

```console
vi /home/hadoop/remove.txt 
node4
```

... and save the file.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Edit the site file</h4>

Open `hdfs-site.xml` and add the following property within the configuration tag. 

```console
<property> 
	<name>dfs.hosts.exclude</name> 
	<value>/home/hadoop/remove.txt</value> 
</property> 

Refresh the cluster to apply the changes in real time:

```console
hdfs dfsadmin -refreshNodes
```


### Refresh

You will observe that the changes will be applied and node4 will change the state from **NORMAL** to **DECOMMISSIONING IN PROGRESS**. 

Once decommissioning is complete, the state will change to **DECOMMISSIONED**. You can track the status either using WebUI of NameNode or by using the following command: 

```console
hdfs dfsadmin -report
```

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png)  Your system output may look a little different


### Whitelisting DataNodes in the Cluster

In this section, we will learn how to whitelist hosts to become DataNodes in the cluster. The intention towards learning this technique is to avoid any random DataNodes from join the cluster. 

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Create a file in the NameNode</h4>

Node0 should hold the set of Hostnames of the nodes that are to be whitelisted. You can set any name to this file. In this case, let us create a file named ‘allow.txt’:

```console
 vi /home/hadoop/allow.txt 
 node2
 node3
 node4
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Open `hdfs-site.xml` and add the following property</h4>

```console
<property>
	<name>dfs.hosts</name> 
	<value>/home/hadoop/allow.txt</value> 
</property>
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>5. Refresh the cluster</h4>

to apply the changes in real time:

```console
hdfs dfsadmin -refreshNodes
```

### Results

Congrats!

You see how the Decommisoning and Whitelisting can be modified, and a new cluster is born!


