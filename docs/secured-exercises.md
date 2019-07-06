## Secured Hadoop Exercises

In this lab we will see how to interact with Hadoop components (HDFS, Hive, Hbase, Sqoop) running on a kerborized cluster and create Ranger appropriate authorization policies for access.

- We will Configure Ranger policies to:
  - Protect /sales HDFS dir - so only sales group has access to it
  - Protect sales hive table - so only sales group has access to it
  - Protect sales HBase table - so only sales group has access to it

#### Access secured HDFS

- Goal: Create a /sales dir in HDFS and ensure only users belonging to sales group (and admins) have access
 
 
- Login to Ranger (using admin/admin) and confirm the HDFS repo was setup correctly in Ranger
  - In Ranger > Under Service Manager > HDFS > Click the Edit icon (next to the trash icon) to edit the HDFS repo
  - Click 'Test connection' 
  - if it fails re-enter below fields and re-try:
    - Username: `rangeradmin@LAB.HORTONWORKS.NET`
    - Password: BadPass#1
    - RPC Protection type: Authentication
  - Once the test passes, click Save  
  
   
- Create /sales dir in HDFS as hadoopadmin
```
#authenticate
sudo -u hadoopadmin kinit
# enter password: BadPass#1

#create dir and set permissions to 000
sudo -u hadoopadmin hdfs dfs -mkdir /sales
sudo -u hadoopadmin hdfs dfs -chmod 000 /sales
```  

- Now login as sales1 and attempt to access it before adding any Ranger HDFS policy
```
sudo su - sales1

hdfs dfs -ls /sales
```
- This fails with `GSSException: No valid credentials provided` because the cluster is kerberized and we have not authenticated yet

- Authenticate as sales1 user and check the ticket
```
kinit
# enter password: BadPass#1

klist
## Default principal: sales1@LAB.HORTONWORKS.NET
```
- Now try accessing the dir again as sales1
```
hdfs dfs -ls /sales
```
- This time it fails with authorization error: 
  - `Permission denied: user=sales1, access=READ_EXECUTE, inode="/sales":hadoopadmin:hdfs:d---------`

- Login into Ranger UI e.g. at http://RANGER_HOST_PUBLIC_IP:6080/index.html as admin/admin

- In Ranger, click on 'Audit' to open the Audits page and filter by below. 
  - Service Type: `HDFS`
  - User: `sales1`
  
- Notice that Ranger captured the access attempt and since there is currently no policy to allow the access, it was "Denied"
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HDFS-denied.png)

- To create an HDFS Policy in Ranger, follow below steps:
  - On the 'Access Manager' tab click HDFS > (clustername)_hadoop
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HDFS-policy.png)
  - This will open the list of HDFS policies
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HDFS-edit-policy.png)
  - Click 'Add New Policy' button to create a new one allowing `sales` group users access to `/sales` dir:
    - Policy Name: `sales dir`
    - Resource Path: `/sales`
    - Group: `sales`
    - Permissions : `Execute Read Write`
    - Add
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HDFS-create-policy.png)

- Wait 30s for policy to take effect
  
- Now try accessing the dir again as sales1 and now there is no error seen
```
hdfs dfs -ls /sales
```

- In Ranger, click on 'Audit' to open the Audits page and filter by below:
  - Service Type: HDFS
  - User: sales1
  
- Notice that Ranger captured the access attempt and since this time there is a policy to allow the access, it was `Allowed`
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HDFS-allowed.png)  

  - You can also see the details that were captured for each request:
    - Policy that allowed the access
    - Time
    - Requesting user
    - Service type (e.g. hdfs, hive, hbase etc)
    - Resource name 
    - Access type (e.g. read, write, execute)
    - Result (e.g. allowed or denied)
    - Access enforcer (i.e. whether native acl or ranger acls were used)
    - Client IP
    - Event count
    
- For any allowed requests, notice that you can quickly check the details of the policy that allowed the access by clicking on the policy number in the 'Policy ID' column
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-policy-details.png)  

- Now let's check whether non-sales users can access the directory

- Logout as sales1 and log back in as hr1
```
kdestroy
#logout as sales1
logout

#login as hr1 and authenticate
sudo su - hr1

kinit
# enter password: BadPass#1

klist
## Default principal: hr1@LAB.HORTONWORKS.NET
```
- Try to access the same dir as hr1 and notice it fails
```
hdfs dfs -ls /sales
## ls: Permission denied: user=hr1, access=READ_EXECUTE, inode="/sales":hadoopadmin:hdfs:d---------
```

- In Ranger, click on 'Audit' to open the Audits page and this time filter by 'Resource Name'
  - Service Type: `HDFS`
  - Resource Name: `/sales`
  
- Notice you can see the history/details of all the requests made for /sales directory:
  - created by hadoopadmin 
  - initial request by sales1 user was denied 
  - subsequent request by sales1 user was allowed (once the policy was created)
  - request by hr1 user was denied
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HDFS-summary.png)  

- Logout as hr1
```
kdestroy
logout
```
- We have successfully setup an HDFS dir which is only accessible by sales group (and admins)

#### Access secured Hive

- Goal: Setup Hive authorization policies to ensure sales users only have access to code, description columns in default.sample_07

- Enable Hive on tez by setting below and restarting Hive 
  - Ambari > Hive > Configs  	
    - Execution Engine = Tez

- Confirm the HIVE repo was setup correctly in Ranger
  - In Ranger > Service Manager > HIVE > Click the Edit icon (next to the trash icon) to edit the HIVE repo
  - Click 'Test connection' 
  - if it fails re-enter below fields and re-try:
    - Username: `rangeradmin@LAB.HORTONWORKS.NET`
    - Password: BadPass#1
  - Once the test passes, click Save  

- Now run these steps from node where Hive (or client) is installed 

- Login as sales1 and attempt to connect to default database in Hive via beeline and access sample_07 table

- Notice that in the JDBC connect string for connecting to an secured Hive while its running in default (ie binary) transport mode :
  - port remains 10000
  - *now a kerberos principal needs to be passed in*

- Login as sales1 without kerberos ticket and try to open beeline connection:
```
sudo su - sales1
kdestroy
beeline -u "jdbc:hive2://localhost:10000/default;principal=hive/$(hostname -f)@LAB.HORTONWORKS.NET"
```
- This fails with `GSS initiate failed` because the cluster is kerberized and we have not authenticated yet

- To exit beeline:
```
!q
```
- Authenticate as sales1 user and check the ticket
```
kinit
# enter password: BadPass#1

klist
## Default principal: sales1@LAB.HORTONWORKS.NET
```
- Now try connect to Hive via beeline as sales1
```
beeline -u "jdbc:hive2://localhost:10000/default;principal=hive/$(hostname -f)@LAB.HORTONWORKS.NET"
```

- If you get the below error, it is because you did not add hive to the global KMS policy in an earlier step (along with nn, hadoopadmin). Go back and add it in.
```
org.apache.hadoop.security.authorize.AuthorizationException: User:hive not allowed to do 'GET_METADATA' on 'testkey'
```

- This time it connects. Now try to run a query
```
beeline> select code, description from sample_07;
```
- Now it fails with authorization error: 
  - `HiveAccessControlException Permission denied: user [sales1] does not have [SELECT] privilege on [default/sample_07]`

- Login into Ranger UI e.g. at http://RANGER_HOST_PUBLIC_IP:6080/index.html as admin/admin

- In Ranger, click on 'Audit' to open the Audits page and filter by below. 
  - Service Type: `Hive`
  - User: `sales1`
  
- Notice that Ranger captured the access attempt and since there is currently no policy to allow the access, it was `Denied`
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HIVE-denied.png)

- To create an HIVE Policy in Ranger, follow below steps:
  - On the 'Access Manager' tab click HIVE > (clustername)_hive
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HIVE-policy.png)
  - This will open the list of HIVE policies
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HIVE-edit-policy.png)
  - Click 'Add New Policy' button to create a new one allowing `sales` group users access to `code` and `description` columns in `sample_07` dir:
    - Policy Name: `sample_07`
    - Hive Database: `default`
    - table: `sample_07`
    - Hive Column: `code` `description`
    - Group: `sales`
    - Permissions : `select`
    - Add
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HIVE-create-policy.png)
  
- Notice that as you typed the name of the DB and table, Ranger was able to look these up and autocomplete them
  -  This was done using the rangeradmin principal we provided during Ranger install

- Wait 30s for the new policy to be picked up
  
- Now try accessing the columns again and now the query works
```
beeline> select code, description from sample_07;
```

- Note though, that if instead you try to describe the table or query all columns, it will be denied - because we only gave sales users access to two columns in the table
  - `beeline> desc sample_07;`
  - `beeline> select * from sample_07;`
  
- In Ranger, click on 'Audit' to open the Audits page and filter by below:
  - Service Type: HIVE
  - User: sales1
  
- Notice that Ranger captured the access attempt and since this time there is a policy to allow the access, it was `Allowed`
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HIVE-allowed.png)  

  - You can also see the details that were captured for each request:
    - Policy that allowed the access
    - Time
    - Requesting user
    - Service type (e.g. hdfs, hive, hbase etc)
    - Resource name 
    - Access type (e.g. read, write, execute)
    - Result (e.g. allowed or denied)
    - Access enforcer (i.e. whether native acl or ranger acls were used)
    - Client IP
    - Event count
    
- For any allowed requests, notice that you can quickly check the details of the policy that allowed the access by clicking on the policy number in the 'Policy ID' column
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HIVE-policy-details.png)  

- Exit beeline
```
!q
```
- Now let's check whether non-sales users can access the table

- Logout as sales1 and log back in as hr1
```
kdestroy
#logout as sales1
logout

#login as hr1 and authenticate
sudo su - hr1

kinit
# enter password: BadPass#1

klist
## Default principal: hr1@LAB.HORTONWORKS.NET
```
- Try to access the same table as hr1 and notice it fails
```
beeline -u "jdbc:hive2://localhost:10000/default;principal=hive/$(hostname -f)@LAB.HORTONWORKS.NET"
```
```
beeline> select code, description from sample_07;
```
- In Ranger, click on 'Audit' to open the Audits page and filter by 'Service Type' = 'Hive'
  - Service Type: `HIVE`

  
- Here you can see the request by sales1 was allowed but hr1 was denied

![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HIVE-summary.png)  

- Exit beeline
```
!q
```
- Logoff as hr1
```
logout
```



- We have setup Hive authorization policies to ensure only sales users have access to code, description columns in default.sample_07


#### Access secured HBase

- Goal: Create a table called 'sales' in HBase and setup authorization policies to ensure only sales users have access to the table
  
- Run these steps from any node where Hbase Master or RegionServer services are installed 

- Login as sales1
```
sudo su - sales1
```
-  Start the hbase shell
```
hbase shell
```
- List tables in default database
```
hbase> list 'default'
```
- This fails with `GSSException: No valid credentials provided` because the cluster is kerberized and we have not authenticated yet

- To exit hbase shell:
```
exit
```
- Authenticate as sales1 user and check the ticket
```
kinit
# enter password: BadPass#1

klist
## Default principal: sales1@LAB.HORTONWORKS.NET
```
- Now try connect to Hbase shell and list tables as sales1
```
hbase shell
hbase> list 'default'
```
- This time it works. Now try to create a table called `sales` with column family called `cf`
```
hbase> create 'sales', 'cf'
```
- Now it fails with authorization error: 
  - `org.apache.hadoop.hbase.security.AccessDeniedException: Insufficient permissions for user 'sales1@LAB.HORTONWORKS.NET' (action=create)`
  - Note: there will be a lot of output from above. The error will be on the line right after your create command

- Login into Ranger UI e.g. at http://RANGER_HOST_PUBLIC_IP:6080/index.html as admin/admin

- In Ranger, click on 'Audit' to open the Audits page and filter by below. 
  - Service Type: `Hbase`
  - User: `sales1`
  
- Notice that Ranger captured the access attempt and since there is currently no policy to allow the access, it was `Denied`
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HBASE-denied.png)

- To create an HBASE Policy in Ranger, follow below steps:
  - On the 'Access Manager' tab click HBASE > (clustername)_hbase
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HBASE-policy.png)
  - This will open the list of HBASE policies
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HBASE-edit-policy.png)
  - Click 'Add New Policy' button to create a new one allowing `sales` group users access to `sales` table in HBase:
    - Policy Name: `sales`
    - Hbase Table: `sales`
    - Hbase Column Family: `*`
    - Hbase Column: `*`
    - Group : `sales`    
    - Permissions : `Admin` `Create` `Read` `Write`
    - Add
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HBASE-create-policy.png)
  
- Wait 30s for policy to take effect
  
- Now try creating the table and now it works
```
hbase> create 'sales', 'cf'
```
  
- In Ranger, click on 'Audit' to open the Audits page and filter by below:
  - Service Type: HBASE
  - User: sales1
  
- Notice that Ranger captured the access attempt and since this time there is a policy to allow the access, it was `Allowed`
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HBASE-allowed.png)  

  - You can also see the details that were captured for each request:
    - Policy that allowed the access
    - Time
    - Requesting user
    - Service type (e.g. hdfs, hive, hbase etc)
    - Resource name 
    - Access type (e.g. read, write, execute)
    - Result (e.g. allowed or denied)
    - Access enforcer (i.e. whether native acl or ranger acls were used)
    - Client IP
    - Event count
    
- For any allowed requests, notice that you can quickly check the details of the policy that allowed the access by clicking on the policy number in the 'Policy ID' column
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HBASE-policy-details.png)  

- Exit hbase shell
```
hbase> exit
```

- Now let's check whether non-sales users can access the table

- Logout as sales1 and log back in as hr1
```
kdestroy
#logout as sales1
logout

#login as hr1 and authenticate
sudo su - hr1

kinit
# enter password: BadPass#1

klist
## Default principal: hr1@LAB.HORTONWORKS.NET
```
- Try to access the same dir as hr1 and notice this user does not even see the table
```
hbase shell
hbase> describe 'sales'
hbase> list 'default'
```

![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-hbase-sales.png)

- Try to create a table as hr1 and it fails with `org.apache.hadoop.hbase.security.AccessDeniedException: Insufficient permissions`
```
hbase> create 'sales', 'cf'
```
- In Ranger, click on 'Audit' to open the Audits page and filter by:
  - Service Type: `HBASE`
  - Resource Name: `sales`

- Here you can see the request by sales1 was allowed but hr1 was denied

![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-HBASE-summary.png)  

- Exit hbase shell
```
hbase> exit
```

- Logout as hr1
```
kdestroy
logout
```
- We have successfully created a table called 'sales' in HBase and setup authorization policies to ensure only sales users have access to the table

- This shows how you can interact with Hadoop components on kerberized cluster and use Ranger to manage authorization policies and audits

<!---
- **TODO: fix for 2.5. Skip for now** At this point your Silk/Banana audit dashboard should show audit data from multiple Hadoop components e.g. http://54.68.246.157:6083/solr/banana/index.html#/dashboard

![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-audit-banana.png)  
--->

#### (Optional) Use Sqoop to import 

- If Sqoop is not already installed, install it via Ambari on same node where Mysql/Hive are installed:
  - Admin > Stacks and Versions > Sqoop > Add service > select node where Mysql/Hive are installed and accept all defaults and finally click "Proceed Anyway"
  - You will be asked to enter admin principal/password:
    - `hadoopadmin@LAB.HORTONWORKS.NET`
    - BadPass#1
  
- *On the host running Mysql*: change user to root and download a sample csv and login to Mysql
```
sudo su - 
wget https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/labdata/PII_data_small.csv
mysql -u root -pBadPass#1
```

- At the `mysql>` prompt run below to: 
  - create a table in Mysql
  - give access to sales1
  - import the data from csv
  - test that table was created
```
create database people;
use people;
create table persons (people_id INT PRIMARY KEY, sex text, bdate DATE, firstname text, lastname text, addresslineone text, addresslinetwo text, city text, postalcode text, ssn text, id2 text, email text, id3 text);
GRANT ALL PRIVILEGES ON people.* to 'sales1'@'%' IDENTIFIED BY 'BadPass#1';
LOAD DATA LOCAL INFILE '~/PII_data_small.csv' REPLACE INTO TABLE persons FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';

select people_id, firstname, lastname, city from persons where lastname='SMITH';
exit
```

- logoff as root
```
logout
```

- Create Ranger policy to allow `sales` group `all permissions` on `persons` table in Hive
  - Access Manager > Hive > (cluster)_hive > Add new policy
  - Create new policy as below and click Add:
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HIVE-create-policy-persons.png) 

- Create Ranger policy to allow `sales` group `all permissions` on `/ranger/audit/kms` dir in HDFS
  - Access Manager > HDFS > (cluster)_hdfs > Add new policy
  - Create new policy as below and click Add:
  **TODO: add screenshot**

  - Log out of Ranger
  
- Create Ranger policy to allow `sales` group `Get Metadata` `GenerateEEK` `DecryptEEK` permissions on `testkey` (i.e. the key used to encrypt Hive warehouse directories)
  - Login to Ranger http://RANGER_PUBLIC_IP:6080 with keyadmin/keyadmin
  - Access Manager > KMS > (cluster)_KMS > Add new policy
  - Create new policy as below and click Add:
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-KMS-create-policy-testkey.png)  
  - Log out of Ranger and re-login as admin/admin

- Login as sales1
```
sudo su - sales1
```

- As sales1 user, kinit and run sqoop job to create persons table in Hive (in ORC format) and import data from MySQL. Below are the details of the arguments passed in:
  - Table: MySQL table name
  - username: Mysql username
  - password: Mysql password
  - hcatalog-table: Hive table name
  - create-hcatalog-table: hive table should be created first
  - driver: classname for Mysql driver
  - m: number of mappers
  
```
kinit
## enter BadPass#1 as password

sqoop import --verbose --connect "jdbc:mysql://$(hostname -f)/people" --table persons --username sales1 --password BadPass#1 --hcatalog-table persons --hcatalog-storage-stanza "stored as orc" -m 1 --create-hcatalog-table  --driver com.mysql.jdbc.Driver
```
- This will start a mapreduce job to import the data from Mysql to Hive in ORC format

- Note: if the mapreduce job fails with below, most likely you have not given sales group all the permissions needed on the EK used to encrypt Hive directories 
```
 java.lang.RuntimeException: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```

- Login to beeline
```
beeline -u "jdbc:hive2://localhost:10000/default;principal=hive/$(hostname -f)@LAB.HORTONWORKS.NET"
```

- Query persons table in beeline
```
beeline> select * from persons;
```
- Since the authorization policy is in place, the query should work

- Ranger audit should show the request was allowed:
  - Under Ranger > Audit > query for
    - Service type: HIVE
![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ranger-HIVE-audit-persons.png)


##### Drop Encrypted Hive tables 

- From beeline, try to drop the persons table. 
```
beeline> drop table persons;
```
- You will get error similar to below
```
message:Unable to drop default.persons because it is in an encryption zone and trash is enabled.  Use PURGE option to skip trash.
```

- To drop a Hive table (when Hive directories are located in EncryptionZone), you need to include `purge` as below:
```
beeline> drop table persons purge;
```

- Destroy the ticket and logout as sales1
```
kdestroy
logout
```

- This completes the lab. You have now interacted with Hadoop components in secured mode and used Ranger to manage authorization policies and audits
