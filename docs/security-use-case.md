### Security Use Case

Use case: Customer has an existing cluster which they would like you to secure for them

- Current setup:
  - The customer has multiple organizational groups (i.e. sales, hr, legal) which contain business users (sales1, hr1, legal1 etc) and hadoopadmin
  - These groups and users are defined in Active Directory (AD) under its own Organizational Unit (OU) called CorpUsers 
  - There are empty OUs created in AD to store hadoop principals/hadoop nodes (HadoopServices, HadoopNodes)
  - Hadoopadmin user has administrative credentials with delegated control of "Create, delete, and manage user accounts" on above OUs
  - Hadoop cluster running HDP has already been setup using Ambari (including HDFS, YARN, Hive, Hbase, Solr, Zookeeper)
  
- Goals:
  - Integrate Ambari with AD - so that hadoopadmin can administer the cluster
  - Integrate Hadoop nodes OS with AD - so business users are recognized and can submit Hadoop jobs
  - Enable kerberos - to secured the cluster and enable authentication
  - Install Ranger and enable Hadoop plugins - to allow admin to setup authorization policies and review audits across Hadoop components
  - Install Ranger KMS and enable HDFS encryption - to be able to create encryption zones
  - Encrypt Hive backing dirs - to protect hive tables
  - Configure Ranger policies to:
    - Protect /sales HDFS dir - so only sales group has access to it
    - Protect sales hive table - so only sales group has access to it
    - Protect sales HBase table - so only sales group has access to it
  - Install Knox and integrate with AD - for perimeter security and give clients access to APIs w/o dealing with kerberos
  - Enable Ambari views to work on secured cluster

We will run through a series of labs and step by step, achieve all of the above goals
  
### AD overview

- Active Directory will already be setup by the instructor. A basic structure of OrganizationalUnits will have been pre-created to look something like the below:
  - CorpUsers OU, which contains:
    - business users and groups (e.g. it1, hr1, legal1) and 
    - hadoopadmin: Admin user (for AD, Ambari, ...)
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/AD-corpusers.png)
  
  - ServiceUsers OU: service users - that would not be created by Ambari  (e.g. rangeradmin, ambari etc)
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/AD-serviceusers.png)
  
  - HadoopServices OU: hadoop service principals (will be created by Ambari)
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/AD-hadoopservices.png)  
  
  - HadoopNodes OU: list of nodes registered with AD
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/AD-hadoopnodes.png)

- In addition, the below steps would have been completed in advance [per doc](http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.2.0/bk_Ambari_Security_Guide/content/_use_an_existing_active_directory_domain.html):
  - Ambari Server and cluster hosts have network access to, and be able to resolve the DNS names of, the Domain Controllers.
  - Active Directory secure LDAP (LDAPS) connectivity has been configured.
  - Active Directory User container for principals has been created and is on-hand. For example, "ou=HadoopServices,dc=lab,dc=hortonworks,dc=net"
  - Active Directory administrative credentials with delegated control of "Create, delete, and manage user accounts" on the previously mentioned User container are on-hand. e.g. hadoopadmin


- For general info on Active Directory refer to Microsoft website [here](https://technet.microsoft.com/en-us/library/hh831484(v=ws.11).aspx) 


### Configure name resolution & certificate to Active Directory

**Run below on all nodes**

1. Add your Active Directory's internal IP to /etc/hosts (if not in DNS). Make sure you replace the IP address of your AD from your instructor below.
  - **Change the IP to match your ADs internal IP**
   ```
ad_ip=GET_THE_AD_IP_FROM_YOUR_INSTRUCTOR
echo "${ad_ip} ad01.lab.hortonworks.net ad01" | sudo tee -a /etc/hosts
   ```

2. Add your CA certificate (if using self-signed & not already configured)
  - In this case we have pre-exported the CA cert from our AD and made available for download. 
   ```
cert_url=https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/extras/ca.crt
sudo yum -y install openldap-clients ca-certificates
sudo curl -sSL "${cert_url}" -o /etc/pki/ca-trust/source/anchors/hortonworks-net.crt

sudo update-ca-trust force-enable
sudo update-ca-trust extract
sudo update-ca-trust check
   ```

3. Test certificate & name resolution with `ldapsearch`

```
## Update ldap.conf with our defaults
sudo tee -a /etc/openldap/ldap.conf > /dev/null << EOF
TLS_CACERT /etc/pki/tls/cert.pem
URI ldaps://ad01.lab.hortonworks.net ldap://ad01.lab.hortonworks.net
BASE dc=lab,dc=hortonworks,dc=net
EOF

##test connection to AD using openssl client
openssl s_client -connect ad01:636 </dev/null

## test connection to AD using ldapsearch (when prompted for password, enter: BadPass#1)
ldapsearch -W -D ldap-reader@lab.hortonworks.net
```

**Make sure to repeat the above steps on all nodes**