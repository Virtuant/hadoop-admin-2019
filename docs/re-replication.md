## Re-Replicate HDFS Files

### Goals

* Let us suppose that you want to decrease the replication factor of your existing 5 node cluster (that you have created earlier) from 3 to 2. 
* We can do the same by modifying hdfs-site.xml without even introducing downtime in the cluster.

**HDFS Files:** `hdfs-site.xml`
 
----

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Edit the config file</h4>

Open `hdfs-site.xml` in node1 (remember where it is?) and add the following property within the configuration tag:

```console
 <property> 
	<name>dfs.replication</name> 
	<value>2</value> 
 </property>
```


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Put the modified config on other nodes</h4>

Copy the modified hdfs-site.xml file in the remaining nodes. 

```console
scp -r /home/hadoop/hadoop2/
```

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png)  Your system output may look a little different

You shoud see soemething like:

```console
etc/hadoop/hdfs-site.xml 
hadoop@node2.mylabs.com:/home/hadoop/hadoop2/etc/hadoop/hdfs-site.xml 
scp -r /home/hadoop/hadoop2/etc/hadoop/hdfs-site.xml 
hadoop@node3.mylabs.com:/home/hadoop/hadoop2/etc/hadoop/hdfs-site.xml 
scp -r /home/hadoop/hadoop2/etc/hadoop/hdfs-site.xml 
hadoop@node4.mylabs.com:/home/hadoop/hadoop2/etc/hadoop/hdfs-site.xml 
```

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Put a file into HDFS</h4>

Try uploading file like

```console
hdfs dfs -mkdir /testdata 
hdfs dfs -copyFromLocal /home/hadoop/sample /testdata/sample 
hdfs dfs -stat %r /testdata/sample 
```

You will see that the file sample is replicated in 2 DataNodes. You can also check the same in WebUI.

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Do again and then change the replication factor of 1 file</h4>

While using this method, we have set the replication factor for all future uploads. All existing files will still hold the same replication factor that was been set during the upload time. Lets take an example to change the replication factor of the existing file:

```console
hdfs dfs -mkdir /data1 
hdfs dfs -put /home/hadoop/sample /data1/sample 
hdfs dfs -stat %r /testdata/sample 
```

The above command will upload the file with RF=2. Now assume you want to change RF to 1. You can do it using `-setrep` command as shown below: 

```console
hdfs dfs -setrep 1 /data1/sample hdfs dfs -stat %r /testdata/
```

### Results

Congrats! Finished!

You see how the replication factor can be modified.
