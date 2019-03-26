## Zookeeper

**Objective**: Practice using Zookeeper to keep track of tables, regions, masters and other information.

**Data directory**: `~/data`

**HDFS paths:** `/user/hadoop`

ZooKeeper is a distributed co-ordination service to manage large set of hosts. Co-ordinating and managing 
a service in a distributed environment is a complicated process. ZooKeeper solves this issue with its simple 

In order to maintain server state in the HBase Cluster, HBase uses ZooKeeper as a distributed coordination service. 
Basically, which servers are alive and available is maintained by Zookeeper, and also it provides server 
failure notification. Moreover, in order to guarantee common shared state, Zookeeper uses consensus.

----

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>1.	Open two command line windows in your lab environment.</h4>

The zookeeper command prompt will be shortened to [zk:] below. Use the HBase `zkcli` to connect to zookeeper and run some queries:

```console
hbase zkcli
[zk: sandbox.hortonworks.com:2181(CONNECTED) 0]
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>2.	The HBase `zkcli` uses commands similar to navigating a file system</h4>

`ls /` will show you the nodes that are children of the root node:

```console
[zk:] ls /
[hbase-unsecure, templeton-hadoop, storm, zookeeper]
```
    
    This shows us that beneath the path, the following child nodes exist: `hbase-unsecure` and `zookeeper`

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>3.	Create  a node</h4>

```console
[zk:] create -e /test 'hello'
```
    
    This creates a node /test with a value of “hello”. The node with -e is an ephemeral node, it will exist as long as the creator of the node is connected to zookeeper.


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>4.	Query zookeeper from the other terminal window to get the value of `/test`</h4>

```console
[zk:] get /test 'hello'
…
```

    This demonstrates that a zookeeper node can be created by one client and seen by any client. This service is the basic need that distributed systems have that zookeeper provides. A consistent distributed information service.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>5.	Delete the ephemeral node:</h4>

```console
[zk:] delete test 
[zk:] ls /
```
    
<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>6.	Create a persistent node</h4>

```console
[zk:] create /test 'hello'
```
    
<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>7.	Quit the command line shell and reconnect</h4>

Notice your node persists if you reconnect or if you get the node from the other connection.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>8. List the HBase zookeeper nodes:</h4>

```console
[zk:] ls /hbase-unsecure
```
	
HBase uses a collection of child nodes under the node `/hbase-unsecure`.

    The result should be

```console
[meta-region-server, backup-masters, region-in-transition, draining, table, running, table-lock, namespace, HBaseid, online-snapshot, replication, splitWAL, recovering- regions, rs]
```
    
    > Note: that the HBase node has many children. If this was a fully distributed cluster there would also be a child node for hmaster. There is a node for meta-region-server which tells client what region server is managing the catalog table meta.
 
<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>9.	Get the value of that node:</h4>

```console
[zk:] get /hbase-unsecure/meta-region-server regionserver:60020ezTSCPBUF
$ sandbox.hortonworks.comъ( cZxid = 0xa47
ctime = Sat Mar 01 21:46:39 PST 2014 mZxid = 0xa47
mtime = Sat Mar 01 21:46:39 PST 2014 pZxid = 0xa47
cversion = 0
dataVersion = 0
aclVersion = 0 ephemeralOwner = 0x0 dataLength = 75
numChildren = 0
```
    
    This shows that it is at `sandbox.hortonworks.com` plus some other information including some unprintable characters.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>10.	 Get the list of tables:</h4>

HBase stores the existing tables as children of the node `/hbase-unsecure/table`.

```console
[zk:] ls /hbase-unsecure/table
```
    
    The result should be:

```console
[timeline, hbase:meta, hbase:namespace, test, cf, ambarismoketest]
```
    
<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>11.	Open another terminal window and launch HBase shell:</h4>

```console
# hbase shell
```
    
    and create another HBase table `zktest`

```console
hbase> create 'zktest','a'
```
    
<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>12. From the zookeeper window, query zookeeper for children </h4>

of the `/hbase-unsecure/table` node:

```console
[zk:] ls /hbase-unsecure/table
[test, ambarismoketest, timeline, hbase:meta, zktest, hbase:namespace, cf]

```
    
    Note the node for zktest in the output.
 
<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>13.	From the HBase window, drop the table in the HBase shell by running:</h4>

```console
hbase> disable 'zktest'
```
    
    Followed by:

```console
hbase> drop 'zktest'
```
    
<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo"> 
<h4>14.	In the HBase zkcli the child nodes of `/hbase-unsecure/table` should no longer show the zktest entry.</h4>


### Results

You should now have been able to use Zookeeper to keep track of tables, regions, masters and other information.


<button type="button"><a href="https://virtuant.github.io/hadoop-overview-spark-hwx/">Go Back</a></button>
<br>
<br>
