## Add HDFS Node in Realtime

### Goals

Hadoop ideally supports horizontal scalability. Here, we will see how to add a node in the existing cluster. 
Ideally, the reason why we would do this is: 

* To increase storage space 
* To increase processing power 

In this lab, we will see how to add the 4th node in the existing 3 node Hadoop cluster. This cluster is running live and active.
 

**HDFS Files:** `hdfs-site.xml`
 
----

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>1. Edit the config file</h4>

Setup network hostname and host file configuration on the 4th machine. Perform the following steps in each machine: 

sudo vi /etc/hostname 
#Replace the existing hostname with the desired hostname as mentioned in previous step. 

For example, 

```console
#for node1 it will be 
node1.mylabs.com 
node5.mylabs.com 
sudo vi /etc/hosts 
#Comment 127.0.1.1 line 
```

and add the following lines in all 4 machines. 

This file holds the information of all machines which needs to be resolved. Each machine will have entries of all 4 machines that are participating in the cluster installation:

```console
52.5.56.157  
53.34.45.143
155.54.76.116
44.73.154.12
```

Once the configuration is done, you will need to restart the machine for hostname changes to take effect. To restart your machine, you can type the following command: 

```console
sudo init 6 
```

You need to also make an entry of the new system in the host file of the first 3 members. Please note that you do not need to restart, since we are only modifying the host file and not touching the hostname file. The intent of this step is to resolve node4 by all machines participating in the cluster.



<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>2. Put the modified config on other nodes</h4>

Setup SSH password-less setup between your NameNode system and the node5.mylabs.com node. In our example setup, we need to setup password-less configuration between node0.mylabs.com and node4.mylabs.com. This is done so that the node0 can contact the node4 to invoke hadoop services. The reason for this kind of configuration is that the NameNode system is the single point of contact for all the users and administrators. 

Perform the following commands on node0: 

```console
ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub hadoop@node4.mylabs.com
```

> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png)  Your system output may look a little different


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>3. Put a file into HDFS</h4>


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>4. Do again and then change the replication factor of 1 file</h4>


<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h4>5. Dynamically Setting up Replication Factor</h4>


> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) the `\` in the command is a continuation 
character. You may leave it out if you wish.


### Results

Congrats! Finished!

You see how the replication factor can be modified.


Step 3: Install Hadoop 2.8.0 in Standalone mode in node5.mylabs.com. Refer Setting up Hadoop-2.8.0 in Standalone Mode (CLI MiniCluster) in the previous chapter in case you require additional help. Step 4: Copy core-site.xml, mapred-site.xml, hdfs-site.xml, and yarn-site.xml from node1.mylabs.com to node5.mylabs.com. On node1.mylabs.com perform the following commands, scp -r /home/hadoop/hadoop2/etc/hadoop/core-site.xml

Prashant Nair. Beginning Apache Hadoop Administration : The First Step towards Hadoop Administration and Management (p. 55). Notion Press, Inc.. Kindle Edition. 

hadoop@node5.mylabs.com:/home/hadoop/hadoop2/etc/hadoop/core-site.xml scp -r /home/hadoop/hadoop2/etc/hadoop/mapred-site.xml hadoop@node5.mylabs.com:/home/hadoop/hadoop2/etc/hadoop/mapred-site.xml scp -r /home/hadoop/hadoop2/etc/hadoop/hdfs-site.xml hadoop@node5.mylabs.com:/home/hadoop/hadoop2/etc/hadoop/hdfs-site.xml scp -r /home/hadoop/hadoop2/etc/hadoop/yarn-site.xml hadoop@node5.mylabs.com:/home/hadoop/hadoop2/etc/hadoop/yarn-site.xml Step 5: Setup Slave configuration in the cluster. We need to perform this step only in Node1. This configuration helps in informing the cluster, which nodes will have Slave services (DataNode and NodeManager). Usually, this configuration is placed only in the NameNode machine. You need

Prashant Nair. Beginning Apache Hadoop Administration : The First Step towards Hadoop Administration and Management (pp. 55-56). Notion Press, Inc.. Kindle Edition. 

to just append node5.mylabs.com on the last line of the file. vi /home/hadoop/hadoop2/etc/hadoop/slaves #Delete localhost and replace the same with node3 and node4 as shown node3.mylabs.com node4.mylabs.com node5.mylabs.com Step 6: Start DataNode and NodeManager service in node5.mylabs.com hadoop-daemon.sh start datanode yarn-daemon.sh start nodemanager Verify whether the services are running or not using ‘jps’ command. Also get the block report to ensure node5 is now a part of the clustr successfully using the following command Step 7: Since we have already scaled the cluster, we will need to balance the load of the cluster.

Prashant Nair. Beginning Apache Hadoop Administration : The First Step towards Hadoop Administration and Management (pp. 56-57). Notion Press, Inc.. Kindle Edition. 

This can be achieved using the following command: On node1.mylabs.com, type the following command start-balancer.sh

Prashant Nair. Beginning Apache Hadoop Administration : The First Step towards Hadoop Administration and Management (p. 57). Notion Press, Inc.. Kindle Edition. 




