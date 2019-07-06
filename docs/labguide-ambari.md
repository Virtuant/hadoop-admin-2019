## Setting Up Ambari

**Linux user**: `centos` **password**: none

**Root access**: `sudo su`   **Root password**: none  


> ![tip](https://user-images.githubusercontent.com/558905/40528496-37bfadac-5fbf-11e8-8b5a-8bea2634f284.png) the user and hostname may be different based on circumstances

----

### Setup with Ambari

Navigate to following URL:
http://sandbox:8080

Log in to the Ambari server using the default credentials admin/admin:






Step 5: Run the Install Wizard
5.1. At the Welcome page, enter the name “horton” for your cluster and click the Next button.
 

5.2. Select the service stack HDP2.1:
 
Step 6: Under “Advanced Repository Options”, uncheck “Red Hat 5” & “SLES 11” and update URL for “Red Hat 6” as mentioned below http://sandbox/hdp/HDP-2.1.2.0
 
6.1. Click “Next” button 


Step 7: Enter the Host and SSH Key Details
7.1. Enter sandbox, node2 and node3 in the list of Target Hosts. (Do not enter node4; you will add that node to the cluster in a later lab.) 
 
7.2. In the Host Registration Information section, click the Choose File button, then browse to and select the training-keypair.pem file (it must be pre-selected, if not you can find it on “Desktop”) :
 

7.3. Click the Register and Confirm button. Click OK if you are warned about not using fully qualified domain names.
Step 8: Confirm Hosts
8.1. Wait for some initial verification to occur on your cluster. Once the process is done, click the Next button to proceed:
 

Step 9: Choose the Services to Install
9.1.  Hortonworks Data Platform is made up of a number of components. You are going to install following services on your cluster initially (Uncheck HBase, Falcon & Storm, we will install them later):
 

9.2. , Click the Next button:
Step 10: Assign Master Nodes
10.1. The Ambari wizard attempts to assign the various master services on appropriate hosts in your cluster. Carefully choose the following assignments of the master services.
CAUTION: Make sure to choose the right node for each master service as specified below. Once the installation starts, you cannot change the selection!
NameNode:	sandbox
SNameNode:	node2
History Server:	node2
App Timeline Server	node2
ResourceManager:	node2
Nagios Server:	node3
Ganglia Server:	node3
HiveServer2:	node2
Oozie Server:	node2
ZooKeeper:	sandbox
ZooKeeper:	node2
ZooKeeper:	node3
10.2. Verify your assignments match the following:
 
10.3. Click the Next button to continue.
Step 11: Assign Slaves and Clients
11.1. Assign all slave and client components to all nodes in the list:
 
11.2. Click the Next button to continue.
Step 12: Customize Services
12.1. Notice three services require additional configuration: Hive, Oozie and Nagios. Click on the Hive tab, then enter hive for the Database Password: 
 

12.2. Click on the Oozie tab and enter oozie for its Database Password:
 
12.3.  Click on the Nagios tab. Enter admin for the Nagios Admin password, and enter your email address in the Hadoop Admin email field:
 
12.4. Click the Next button to continue.
Step 13: Review the Configuration
13.1. Notice the Review page allows you to review your complete install configuration. If you’re satisfied that everything is correct, click Deploy to start the installation process. (If you need to go back and make changes, you can use the Back button.)
 
Step 14: Wait for HDP to Install
14.1. The installation will begin now. It will take up to 30 minutes to complete, depending on your computer processor and allocated RAM. You will see progress updates under the Status column as components are installed, tested, and started:
 
14.2. You should see the following screen if the installation completes successfully:
 
14.3. When the process completes, click Next to get a summary of the installation process.  Check all configured services are on the expected nodes, then click Complete:
 
Step 15: View the Ambari Dashboard
15.1. After the install wizard completes, you will be directed to your cluster’s Ambari Dashboard page. Verify the DataNodes Live status shows 3/3:

 
RESULT: You now have a running 3-node cluster of the Hortonworks Data Platform!
