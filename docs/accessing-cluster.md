## Accessing your Cluster

Credentials will be provided for these services by the instructor:

* SSH
* Ambari

## Use your Cluster

### To connect using Putty from Windows laptop

- Right click to download [this ppk key](https://github.com/HortonworksUniversity/Security_Labs/raw/master/training-keypair.ppk) > Save link as > save to Downloads folder
- Use putty to connect to your node using the ppk key:
  - Connection > SSH > Auth > Private key for authentication > Browse... > Select training-keypair.ppk
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/putty.png)

- Make sure to click "Save" on the session page before logging in
- When connecting, it will prompt you for username. Enter `centos`

### To connect from Linux/MacOSX laptop

- SSH into Ambari node of your cluster using below steps:
- Right click to download [this pem key](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/training-keypair.pem)  > Save link as > save to Downloads folder
  - Copy pem key to ~/.ssh dir and correct permissions
  ```
  cp ~/Downloads/training-keypair.pem ~/.ssh/
  chmod 400 ~/.ssh/training-keypair.pem
  ```
 - Login to the Ambari node of the cluster you have been assigned by replacing IP_ADDRESS_OF_AMBARI_NODE below with Ambari node IP Address (your instructor will provide this)   
  ```
  ssh -i  ~/.ssh/training-keypair.pem centos@IP_ADDRESS_OF_AMBARI_NODE
  ```
  - To change user to root you can:
  ```
  sudo su -
  ```

- Similarly login via SSH to each of the other nodes in your cluster as you will need to run commands on each node in a future lab

- Tip: Since in the next labs you will be required to run *the same set of commands* on each of the cluster hosts, now would be a good time to setup your favorite tool to do so: examples [here](https://www.reddit.com/r/sysadmin/comments/3d8aou/running_linux_commands_on_multiple_servers/)
  - On OSX, an easy way to do this is to use [iTerm](https://www.iterm2.com/): open multiple tabs/splits and then use 'Broadcast input' feature (under Shell -> Broadcast input)
  - If you are not already familiar with such a tool, you can also just run the commands on the cluster, one host at a time

#### Login to Ambari

- Login to Ambari web UI by opening http://AMBARI_PUBLIC_IP:8080 and log in with admin/BadPass#1

- You will see a list of Hadoop components running on your cluster on the left side of the page
  - They should all show green (ie started) status. If not, start them by Ambari via 'Service Actions' menu for that service

#### Finding internal/external hosts

- Following are useful techniques you can use in future labs to find your cluster specific details:

  - From SSH terminal, how can I find the cluster name?
  ```
  #run on ambari node to fetch cluster name via Ambari API
  PASSWORD=BadPass#1
  output=`curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari'  http://localhost:8080/api/v1/clusters`
  cluster=`echo $output | sed -n 's/.*"cluster_name" : "\([^\"]*\)".*/\1/p'`
  echo $cluster
  ```
  - From SSH terminal, how can I find internal hostname (aka FQDN) of the node I'm logged into?
  ```
  $ hostname -f
  ip-172-30-0-186.us-west-2.compute.internal  
  ```

  - From SSH terminal, how can I to find external hostname of the node I'm logged into?
  ```
  $ curl icanhazptr.com
  ec2-52-33-248-70.us-west-2.compute.amazonaws.com 
  ```

  - From SSH terminal, how can I to find external (public) IP  of the node I'm logged into?
  ```
  $ curl icanhazip.com
  54.68.246.157  
  ```
  
  - From Ambari how do I check the cluster name?
    - It is displayed on the top left of the Ambari dashboard, next to the Ambari logo. If the name appears truncated, you can hover over it to produce a helptext dialog with the full name
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/clustername.png)
  
  - From Ambari how can I find external hostname of node where a component (e.g. Resource Manager or Hive) is installed?
    - Click the parent service (e.g. YARN) and *hover over* the name of the component. The external hostname will appear.
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-RM-public-host.png)  

  - From Ambari how can I find internal hostname of node where a component (e.g. Resource Manager or Hive) is installed?
    - Click the parent service (e.g. YARN) and *click on* the name of the component. It will take you to hosts page of that node and display the internal hostname on the top.
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-YARN-internal-host.png)  
  
  - In future labs you may need to provide private or public hostname of nodes running a particular component (e.g. YARN RM or Mysql or HiveServer)
  
  
#### Import sample data into Hive 


- Run below *on the node where HiveServer2 is installed* to download data and import it into a Hive table for later labs
  - You can either find the node using Ambari as outlined in Lab 1
  - Download and import data
  ```
  cd /tmp
  wget https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/labdata/sample_07.csv
  wget https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/labdata/sample_08.csv
  ```
  - Create user dir for admin, sales1 and hr1
  ```
   sudo -u hdfs hdfs dfs  -mkdir /user/admin
   sudo -u hdfs hdfs dfs  -chown admin:hadoop /user/admin

   sudo -u hdfs hdfs dfs  -mkdir /user/sales1
   sudo -u hdfs hdfs dfs  -chown sales1:hadoop /user/sales1
   
   sudo -u hdfs hdfs dfs  -mkdir /user/hr1
   sudo -u hdfs hdfs dfs  -chown hr1:hadoop /user/hr1   
  ```
    
  - Now create Hive table in default database by 
    - Start beeline shell from the node where Hive is installed: 
```
beeline -n admin -u "jdbc:hive2://localhost:10000/default"
```

  - At beeline prompt, run below:
    
```
CREATE TABLE `sample_07` (
`code` string ,
`description` string ,  
`total_emp` int ,  
`salary` int )
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TextFile;
```
```
load data local inpath '/tmp/sample_07.csv' into table sample_07;
```
```
CREATE TABLE `sample_08` (
`code` string ,
`description` string ,  
`total_emp` int ,  
`salary` int )
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TextFile;
```
```
load data local inpath '/tmp/sample_08.csv' into table sample_08;
```

- Notice that in the JDBC connect string for connecting to an unsecured Hive while its running in default (ie binary) transport mode :
  - port is 10000
  - no kerberos principal was needed 

- This will change after we:
  - enable kerberos
  - configure Hive for http transport mode (to go through Knox)
    
### Why is security needed?


##### HDFS access on unsecured cluster

- On your unsecured cluster try to access a restricted dir in HDFS
```
hdfs dfs -ls /tmp/hive   
## this should fail with Permission Denied
```

- Now try again after setting HADOOP_USER_NAME env var
```
export HADOOP_USER_NAME=hdfs
hdfs dfs -ls /tmp/hive   
## this shows the file listing!
```
- Unset the env var and it will fail again
```
unset HADOOP_USER_NAME
hdfs dfs -ls /tmp/hive  
```

##### WebHDFS access on unsecured cluster

- From *node running NameNode*, make a WebHDFS request using below command:
```
curl -sk -L "http://$(hostname -f):50070/webhdfs/v1/user/?op=LISTSTATUS"
```

- In the absence of Knox, notice it goes over HTTP (not HTTPS) on port 50070 and no credentials were needed

##### Web UI access on unsecured cluster

- From Ambari notice you can open the WebUIs without any authentication
  - HDFS > Quicklinks > NameNode UI
  - Madreduce > Quicklinks > JobHistory UI
  - YARN > Quicklinks > ResourceManager UI
    
- This should tell you why kerberos (and other security) is needed on Hadoop :)


### Install Additional Components

#### Install Knox via Ambari

- Login to Ambari web UI by opening http://AMBARI_PUBLIC_IP:8080 and log in with admin/BadPass#1
- Use the 'Add Service' Wizard (under 'Actions' dropdown, near bottom left of page) to install Knox *on a node other than the one running Ambari*
  - **Make sure not to install Knox on same node as Ambari** (or if you must, change its port from 8443)
    - Reason: in a later lab after we enable SSL for Ambari, it will run on port 8443
  - When prompted for the `Knox Master Secret`, set it to `knox`
  - Do *not* use password with special characters (like #, $ etc) here as seems beeline may have problem with it
   ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-Knox-install.png)
  - Click Next > Proceed Anyway > Deploy to accept all defaults

- We will use Knox further in a later exercise.
  
- After the install completed, Ambari will show that a number of services need to be restarted. Ignore this for now, we will restart them at a later stage.

#### Install Tez on Pig nodes

- Ensure Tez is installed on all nodes where Pig clients are installed. This is done to ensure Pig service checks do not fail later on.
 - Ambari > Pig > click the 'Pig clients' link
 - This tell us which nodes have Pig clients installed
   ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-pig-nodes.png)
 - For each node that has Pig installed:
   - Click on the hyperlink of the node name to view that shows all the services running on that particular node
   - Click '+Add' and select 'Tez client' > Confirm add 
     - If 'Tez client'does not appear in the list, it is already installed on this host, so you can skip this host
   ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-host-add-tez.png)   
