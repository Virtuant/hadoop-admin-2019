## Objective:
As an alternative to the Ambari-oriented installation, to perform an installation of an HBase cluster based on manual installation instructions using RPM files

### File locations:

  ```console 
  /etc/hbase/conf/hbase_env.sh
  /etc/hbase/conf/hbase-site.xml
  /etc/zookeeper/conf/zookeeper_env.sh
  /etc/zookeeper/conf/zoo.cg
  ```

### Successful outcome:
You will:
Manually install a working ZooKeeper quorum and HBase cluster by configuring a three-node quorum of Zookeeper servers to be used by HBase; Installing, configuring and starting an HBase master server; and Installing, configuring and starting three HBase regionservers

### Lab Steps

Complete the setup guide

1. The Virtual Machine (VM) is an Ubuntu Linux machine that hosts five Docker containers that emulate a multimode Hadoop cluster. Table 1 shows how Hadoop and HBase services are distributed across the five nodes.

  |Node Name| Services Hosted|
  |--|--|
  |node1|	Namenode, Resourcemanager, Zookeeper Server, Regionserver|
  |node2	|Datanode,  Nodemanager, Regionserver|
  |node3	|Datanode,  Nodemanager, Regionserver|
  |node4	|Datanode, Nodemanager, HBase master, Regionserver|
  |node5	|Reserved for standalone HBase master for replication lab|


2. Confirm HDFS Storage Capability
HBase requires HDFS storage capability, which has already been installed and configured for you. Confirm you have a running HDFS cluster.

a.	Open Firefox by clicking on the Firefox icon on the left side of your screen.
# IMAGE GOES HERE

c.	Navigate to http://node1:50070/

d.	Click on the Datanodes tab to view the list of DataNodes.

3. Configure Zookeeper
HBase requires a Zookeeper quorum as a metadata store. Zookeeper has already been installed on your cluster, but it is neither configured nor running. Configure Zookeeper by editing configuration files on node1 and then copying those files to node2 and node3.

a.	Open a terminal window by clicking on the terminal icon on the left side of your screen.

# IMAGE GOES HERE

b.	ssh to node1

a.	Using vi or nano, edit `/etc/zookeeper/conf/zookeeper-env.sh`
i.	Add these lines:

  ```console
  export ZOO_LOG_DIR=/var/log/zookeeper
  export ZOOPIDFILE=/var/run/zookeeper/zookeeper-server.pid 
  export ZOOKEEPER_USER=zookeeper
  export ZOO_DATA_DIR=/hadoop/zookeeper/data
  ```
  
Do not modify JAVA_HOME definition. The root shell environment already has JAVA_HOME set.
4	. Create a directory for zookeeper data
```console
  [root@node1]# mkdir -p /hadoop/zookeeper/data
  [root@node1]# chown zookeeper:hadoop /hadoop/zookeeper/data 
  [root@node1]# chmod 755 /hadoop/zookeeper/data
 
  ```
a.	Edit /etc/zookeeper/conf/zoo.cfg
i.	Add these lines to the bottom of the file.
```console
   dataDir=/hadoop/zookeeper/data 
   server.1=node1:2888:3888 
   server.2=node2:2888:3888 
   server.3=node3:2888:3888
   ```
5	. Start Zookeeper on node1
a.	Each Zookeeper server needs an ID that makes it unique from other servers in the quorum.
Create an ID file for Zookeeper server.
```
  [root@node1]# echo '1' > /hadoop/zookeeper/data/myid	
  ```
b.	Start zookeeper on node1 and watch for errors.

```
  [root@node1]# su - zookeeper -c "/usr/hdp/current/zookeeper- 
  client/bin/zkServer.sh start"
  JMX enabled by default Using config: /usr/hdp/current/zookeeper- 
  client/bin/../conf/zoo.cfg Starting zookeeper ... STARTED
  ```

c.	Use the nc command to connect to the local Zookeeper server. Echo the zookeeper four-letter ruok (“are you okay”) command.
```
  [root@node1] # echo ruok | nc localhost 2181 imok[root@node1]
 ```[root@node1] # echo ruok | nc localhost 2181 
  imok[root@node1]
 ```

d.	Look for the imok ("I am okay") response immediately before the command line prompt (as shown above).
6. Set up Zookeeper servers on node2 and node3.
a.	Copy the modified configuration files from node1 to node2 and node3
```
  [root@node1]# scp zookeeper-env.sh node2:/etc/zookeeper/conf
  zookeeper-env.sh 100% 982 1.0KB/s 00:00 [root@node1]# scp 
  zoo.cfg node2:/etc/zookeeper/conf
  zoo.cfg 100% 1281 1.3KB/s 00:00 [root@node1]# scp 
  zookeeper-env.sh node3:/etc/zookeeper/conf
  zookeeper-env.sh 100% 982 1.0KB/s 00:00 [root@node1]# scp 
  zoo.cfg node3:/etc/zookeeper/conf
  zoo.cfg 100% 1281 1.3KB/s 00:00
  ```
  b. ssh to node2 and complete the configuration.
  
 ```
  [root@node2]# mkdir -p /hadoop/zookeeper/data
  [root@node2]# chown zookeeper:hadoop /hadoop/zookeeper/data 
  [root@node2]# chmod 755 /hadoop/zookeeper/data 
  [root@node2]# echo '2' >/hadoop/zookeeper/data/myid
  ```
  Start Zookeeper on node2 and node3
  Start Zookeeper on node2 
```
  [root@node2]# su - zookeeper -c "/usr/hdp/current/zookeeper- 
  client/bin/zkServer.sh start"
  JMX enabled by default
  Using config: /usr/hdp/current/zookeeper-client/bin/../conf/zoo.cfg 
  Starting zookeeper ... STARTED
  ```
  Confirm Zookeeper's status using the stat command.
  ```
    [root@node2]# echo 'stat' | nc localhost 2181
    Zookeeper version: 3.4.6-2--1, built on 03/31/2015 19:31 GMT 
    Clients:
    /127.0.0.1:40860[0](queued=0,recved=1,sent=0)
    Latency min/avg/max: 0/0/0
    Received: 1
    Sent: 0
    Connections: 1
    Outstanding: 0
    Zxid: 0x100000000
    Mode: follower
    Node count: 4
    ```
  NOTE:Modein the output of the stat command will show “follower” or “leader”, if Zookeeper server is properly configured to     create a quorum with other Zookeeper servers. ssh to node3, complete the configuration, start Zookeeper, and confirm its       status.
  
 ```
    [root@node3]# mkdir -p /hadoop/zookeeper/data
    [root@node3]# chown zookeeper:hadoop /hadoop/zookeeper/data 
    [root@node3]# chmod 755 /hadoop/zookeeper/data 
    [root@node3]# echo '3' > /hadoop/zookeeper/data/myid
    [root@node3]# su - zookeeper -c "/usr/hdp/current/zookeeper- 
    client/bin/zkServer.sh start"
    JMX enabled by default
    Using config: /usr/hdp/current/zookeeper-client/bin/../conf/zoo.cfg 
    Starting zookeeper ... STARTED
    [root@node3]# echo 'stat' | nc localhost 2181
    Zookeeper version: 3.4.6-2--1, built on 03/31/2015 19:31 GMT 
    Clients:
    /127.0.0.1:40860[0](queued=0,recved=1,sent=0)
    Latency min/avg/max: 0/0/0
    Received: 1
    Sent: 0
    Connections: 1
    Outstanding: 0
    Zxid: 0x100000000
    Mode: follower
    Node count: 4
    ```
    
7 .Install HBase software on all nodes.
With a working Zookeeper quorum, install HBase software on node4, where the HBase master server will be running.
Then install HBase on the other nodes, which will be region servers.
a. ssh to node4
b. Install the HBase software from the rpm file.

```
   [root@node4]# yum install –y hbase
   ```
Repeat on node1, node2 and node3.
8 . Configure the shell environment for running master and regionservers
a. ssh to node4
b. Edit /etc/hbase/conf/hbase-env.sh
9 . Remove the comment character “#” from the beginning of this line:

```
  # export SERVER_GC_OPTS="-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps 
  -Xloggc:<FILE-PATH> -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=1 - 
  XX:GCLogFileSize=512M"
  ```
  Should be:
```
  export SERVER_GC_OPTS="-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps - 
  Xloggc:<FILE-PATH> -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=1 - 
  XX:GCLogFileSize=512M"
  ```

a. Uncomment this line:
```
  # export HBASE_REGIONSERVERS=${HBASE_HOME}/conf/regionservers
  ```
a.	Edit the line to match as follows:
```
  export HBASE_REGIONSERVERS=/etc/hbase/conf/regionservers
  ```
b.	Add these lines to the bottom of the file:
```
  export HBASE_LOG_DIR=/var/log/hbase 
  export HBASE_PID_DIR=/var/run/hbase
  ```
 

10	. Provide a list of regionservers to the HBase master server
HBase master server needs a list of regionservers from which to accept connections. Edit the /etc/hbase/conf/regionservsers file to contain these three lines:

```
  node1
  node2
  node3
  ```
  
11	. Confirm HBase log and pid directories
The yum installation created the HBase log and pid directories. Confirm their existence, their ownership, and their permission modes.
```
  [root@node4 conf]# ls -ld /var/log/hbase
  drwxrwxr-x 2 hbase hbase 4096 Mar 31 21:19 /var/log/hbase 
  [root@node4 conf]# ls -ld /var/run/hbase
  drwxrwxr-x 2 hbase hbase 4096 Mar 31 21:19 /var/run/hbase
  ```
12	. Edit /etc/hbase/conf/hbase-site.xml
a.	These lines:
```
  <property>
       <name>hbase.rootdir</name> 
       <value>hdfs://node1:8020/apps/hbase</value>
  </property>
  ```

b.	Establish the Zookeeper quorum configuratio.
``` 
  <property>
       <name>hbase.rootdir</name> 
       <value>hdfs://node1:8020/apps/hbase</value>
  </property>
  ```

c.	Configure HBase master server to operate in a distributed mode.
```
  <property> 
       <name>hbase.cluster.distributed</name> 
       <value>true</value>
  </property>
  ```

13	. Create the HBase data directory in HDFS.
a.	This function requires superuser authority on the HDFS cluster. Switch user to hdfs.
[root@node4]# su – hdfs	
```
  [root@node4]# su – hdfs
  ```

b.	Create the /apps/hbase/data directory and properly set ownership.
```
  -bash-4.1$ hadoop fs -mkdir -p /apps/hbase/data 
  -bash-4.1$ hadoop fs -chown -R hbase:hbase /apps/hbase 
  -bash-4.1$ exit
  logout
  [root@node4]#
  ```

14	. Start the HBase master.
```
  [root@node4]# su - hbase -c "/usr/hdp/current/hbase-master/bin/hbase-daemon.sh 
  start master"
  starting master, logging to /var/log/hbase/hbase-hbase-master-node4.out
 ```

15	. Validate the status of the HBase master by checking the browser-based UI. Open Firefox and browse to http://node4:60010/.

#IMAGE HERE

16	. Copy the configuration files to the other three nodes, where regionservers will then be started.
```
  [root@node4]# /etc/hbase/conf
  [root@node4]# scp hbase-env.sh node1:/etc/hbase/conf 
  hbase-env.sh 100% 7238 7.1KB/s 00:00
  [root@node4]# scp hbase-env.sh node2:/etc/hbase/conf 
  hbase-env.sh 100% 7238 7.1KB/s 00:00
  [root@node4]# scp hbase-env.sh node3:/etc/hbase/conf 
  hbase-env.sh 100% 7238 7.1KB/s 00:00
  [root@node4]# scp hbase-site.xml node1:/etc/hbase/conf 
  hbase-site.xml 100% 1263 1.2KB/s 00:00
  [root@node4]# scp hbase-site.xml node2:/etc/hbase/conf 
  hbase-site.xml 100% 1263 1.2KB/s 00:00
  [root@node4]# scp hbase-site.xml node3:/etc/hbase/conf 
  hbase-site.xml 100% 1263 1.2KB/s 00:00
  ```

17	. For each regionserver node (node1, node2 and node3), login and start the regionserver.
```
  [root@node1]# su - hbase -c "/usr/hdp/current/hbase-master/bin/hbase-daemon.sh 
  start regionserver"
  [root@node2]# su - hbase -c "/usr/hdp/current/hbase-master/bin/hbase-daemon.sh 
  start regionserver"
  [root@node3]# su - hbase -c "/usr/hdp/current/hbase-master/bin/hbase-daemon.sh 
  start regionserver"
  ```

18	. Return to the Master server UI, and confirm that all four regionservers are running.

#IMAGE HERE


Result
You have now:
Manually installed a working ZooKeeper quorum and HBase cluster by configuring a three-node quorum of Zookeeper servers to be used by HBase; Installing, configuring and starting an HBase master server; and Installing, configuring and starting three HBase regionservers.

 
 
  






