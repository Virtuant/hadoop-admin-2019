## Ranger install

Goal: In this lab we will install Apache Ranger via Ambari and setup Ranger plugins for Hadoop components: HDFS, Hive, Hbase, YARN, Knox. We will also enable Ranger audits to Solr and HDFS

### Ranger prereqs

##### Create & confirm MySQL user 'root'

Prepare MySQL DB for Ranger use.

- Run these steps on the node where MySQL/Hive is located. To find this, you can either:
  - use Ambari UI or
  - Just run `mysql` on each node: if it returns `mysql: command not found`, move onto next node

- `sudo mysql`
- Execute following in the MySQL shell. Change the password to your preference. 

```sql
CREATE USER 'root'@'%';
GRANT ALL PRIVILEGES ON *.* to 'root'@'%' WITH GRANT OPTION;
SET PASSWORD FOR 'root'@'%' = PASSWORD('BadPass#1');
SET PASSWORD = PASSWORD('BadPass#1');
FLUSH PRIVILEGES;
exit
```

- Confirm MySQL user: `mysql -u root -h $(hostname -f) -p -e "select count(user) from mysql.user;"`
  - Output should be a simple count. Check the last step if there are errors.

##### Prepare Ambari for MySQL
- Run this on Ambari node
- Add MySQL JAR to Ambari:
  - `sudo ambari-server setup --jdbc-db=mysql --jdbc-driver=/usr/share/java/mysql-connector-java.jar`
    - If the file is not present, it is available on RHEL/CentOS with: `sudo yum -y install mysql-connector-java`

<---
##### Install SolrCloud from HDPSearch for Audits (if not already installed)

This should already be installed on your cluster. If not, refer to appendix [here](https://github.com/HortonworksUniversity/Security_Labs#install-solrcloud)


###### Setup Solr for Ranger audit 

- Starting HDP 2.5, if you have deployed Logsearch/Ambari Infra services, you can just use the embedded Solr for Ranger audits.
  - Just make sure Logsearch is installed/started and proceed

- **TODO**: add steps to install/configure Banana dashboard for Ranger Audits
--->

## Ranger install

##### Install Ranger

- Start the Ambari 'Add Service' wizard and select Ranger

- When prompted for where to install it, choose any node you like

- On the Ranger Requirements popup windows, you can check the box and continue as we have already completed the pre-requisite steps

- On the 'Customize Services' page of the wizard there are a number of tabs that need to be configured as below

- Go through each Ranger config tab, making below changes:

1. Ranger Admin tab:
  - Ranger DB Host = FQDN of host where Mysql is running (e.g. ip-172-30-0-242.us-west-2.compute.internal)
  - Enter passwords: BadPass#1
  - Click 'Test Connection' button
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-1.png)
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-2.png)

2. Ranger User info tab
  - 'Sync Source' = LDAP/AD 
  - Common configs subtab
    - Enter password: BadPass#1
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-3.png)
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-3.5.png)

3. Ranger User info tab 
  - User configs subtab
    - User Search Base = `ou=CorpUsers,dc=lab,dc=hortonworks,dc=net`
    - User Search Filter = `(objectcategory=person)`
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-4.png)
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-5.png)

4. Ranger User info tab 
  - Group configs subtab
    - Make sure Group sync is disabled
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-6.png)

5. Ranger plugins tab
  - Enable all plugins
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-7.png)

6. Ranger Audits tab 
  - SolrCloud = ON
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-8.png)
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-9.png)

7.Advanced tab
  - No changes needed (skipping configuring Ranger authentication against AD for now)
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/ali/ranger-213-setup/ranger-213-10.png)

- Click Next > Proceed Anyway to proceed
    
- If prompted, on Configure Identities page, you may have to enter your AD admin credentials:
  - Admin principal: `hadoopadmin@LAB.HORTONWORKS.NET`
  - Admin password: BadPass#1
  - Notice that you can now save the admin credentials. Check this box too
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-configureidentities.png)
  
- Click Next > Deploy to install Ranger

- Once installed, restart components that require restart (e.g. HDFS, YARN, Hive etc)

- (Optional) In case of failure (usually caused by incorrectly entering the Mysql nodes FQDN in the config above), delete Ranger service from Ambari and retry.



8 - (Optional) Enable Deny condition in Ranger 

The deny condition in policies is optional by default and must be enabled for use.

- From Ambari>Ranger>Configs>Advanced>Custom ranger-admin-site, add : 
`ranger.servicedef.enableDenyAndExceptionsInPolicies=true`

- Restart Ranger

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_security/content/about_ranger_policies.html


##### Check Ranger

- Open Ranger UI at http://RANGERHOST_PUBLIC_IP:6080 using admin/admin
- Confirm that repos for HDFS, YARN, Hive, HBase, Knox appear under 'Access Manager tab'
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-AccessManager.png)

- Confirm that audits appear under 'Audit' > 'Access' tab
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-audits.png)

  - If audits do not show up here, you may need to restart Solr from Ambari
  
- Confirm that plugins for HDFS, YARN, Hive etc appear under 'Audit' > 'Plugins' tab 
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-plugins.png)

- Confirm users/group sync from AD into Ranger are working by clicking 'Settings' > 'Users/Groups tab' in Ranger UI and noticing AD users/groups are present
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-user-groups.png)

- Confirm HDFS audits working by querying the audits dir in HDFS:

```
#### 1 authenticate
export PASSWORD=BadPass#1

#detect name of cluster
output=`curl -u hadoopadmin:$PASSWORD -k -i -H 'X-Requested-By: ambari'  https://localhost:8443/api/v1/clusters`
cluster=`echo $output | sed -n 's/.*"cluster_name" : "\([^\"]*\)".*/\1/p'`

echo $cluster
## this should show the name of your cluster

## if not you can manully set this as below
## cluster=Security-HWX-LabTesting-XXXX

#then kinit as hdfs using the headless keytab and the principal name
sudo -u hdfs kinit -kt /etc/security/keytabs/hdfs.headless.keytab "hdfs-${cluster,,}"
    
#### 2 read audit dir in hdfs 
sudo -u hdfs hdfs dfs -cat /ranger/audit/hdfs/*/*
```

<!---
- Confirm Solr audits working by querying Solr REST API *from any solr node* - SKIP 
```
curl "http://localhost:6083/solr/ranger_audits/select?q=*%3A*&df=id&wt=csv"
```

- Confirm Banana dashboard has started to show HDFS audits - SKIP
http://PUBLIC_IP_OF_SOLRLEADER_NODE:6083/solr/banana/index.html#/dashboard

![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Banana-audits.png)
--->