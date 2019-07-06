## Kerberize the Cluster

### Run Ambari Kerberos Wizard against Active Directory environment

- Enable kerberos using Ambari security wizard (under 'Admin' tab > Kerberos > Enable kerberos > proceed). 
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-start-kerberos-wizard.png)

- Select "Existing Active Directory" and check all the boxes
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-1.png)
  
- Enter the below details:

- KDC:
    - KDC host: `ad01.lab.hortonworks.net`
    - Realm name: `LAB.HORTONWORKS.NET`
    - LDAP url: `ldaps://ad01.lab.hortonworks.net`
    - Container DN: `ou=HadoopServices,dc=lab,dc=hortonworks,dc=net`
    - Domains: `us-west-2.compute.internal,.us-west-2.compute.internal`
- Kadmin:
    - Kadmin host: `ad01.lab.hortonworks.net`
    - Admin principal: `hadoopadmin@LAB.HORTONWORKS.NET`
    - Admin password: `BadPass#1`

  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-2.png)
  - Notice that the "Save admin credentials" checkbox is available, clicking the check box will save the "admin principal".
  - Sometimes the "Test Connection" button may fail (usually related to AWS issues), but if you previously ran the "Configure name resolution & certificate to Active Directory" steps *on all nodes*, you can proceed.
  
- Now click Next on all the following screens to proceed with all the default values  

  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-3.png)

  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-4.png)

  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-5.png)

  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-6.png)

  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-7.png)

  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-kerberos-wizard-8.png)

  - Note if the wizard fails after completing more than 90% of "Start and test services" phase, you can just click "Complete" and manually start any unstarted services (e.g. WebHCat or HBase master)


- Check the keytabs directory and notice that keytabs have been generated here:
```
ls -la /etc/security/keytabs/
```

- Run a `klist -ekt`  one of the service keytab files to see the principal name it is for. Sample output below (*executed on host running Namenode*):
```
$ sudo klist -ekt /etc/security/keytabs/nn.service.keytab
Keytab name: FILE:/etc/security/keytabs/nn.service.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   0 10/03/2016 22:20:12 nn/ip-172-30-0-181.us-west-2.compute.internal@LAB.HORTONWORKS.NET (des3-cbc-sha1)
   0 10/03/2016 22:20:12 nn/ip-172-30-0-181.us-west-2.compute.internal@LAB.HORTONWORKS.NET (arcfour-hmac)
   0 10/03/2016 22:20:12 nn/ip-172-30-0-181.us-west-2.compute.internal@LAB.HORTONWORKS.NET (des-cbc-md5)
   0 10/03/2016 22:20:12 nn/ip-172-30-0-181.us-west-2.compute.internal@LAB.HORTONWORKS.NET (aes128-cts-hmac-sha1-96)
   0 10/03/2016 22:20:12 nn/ip-172-30-0-181.us-west-2.compute.internal@LAB.HORTONWORKS.NET (aes256-cts-hmac-sha1-96)
```

- Notice how the service keytabs are divided into the below 3 parts. The instance here is the FQDN of the node so these keytabs are *host specific*.
```
{name of entity}/{instance}@{REALM}. 
```

- Run a `klist -kt`  on one of the headless keytab files to see the principal name it is for. Sample output below (*executed on host running Namenode*):
```
$ sudo klist -ekt /etc/security/keytabs/hdfs.headless.keytab
Keytab name: FILE:/etc/security/keytabs/hdfs.headless.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   0 10/03/2016 22:20:12 hdfs-Security-HWX-LabTesting-100@LAB.HORTONWORKS.NET (des3-cbc-sha1)
   0 10/03/2016 22:20:12 hdfs-Security-HWX-LabTesting-100@LAB.HORTONWORKS.NET (arcfour-hmac)
   0 10/03/2016 22:20:12 hdfs-Security-HWX-LabTesting-100@LAB.HORTONWORKS.NET (des-cbc-md5)
   0 10/03/2016 22:20:12 hdfs-Security-HWX-LabTesting-100@LAB.HORTONWORKS.NET (aes128-cts-hmac-sha1-96)
   0 10/03/2016 22:20:12 hdfs-Security-HWX-LabTesting-100@LAB.HORTONWORKS.NET (aes256-cts-hmac-sha1-96)
```

- Notice how the headless keytabs are divided into the below 3 parts. These keytabs are *cluster specific* (i.e one per cluster)
```
{name of entity}-{cluster}@{REALM}. 
```

### Setup AD/OS integration via SSSD

- Why? 
  - Currently your hadoop nodes do not recognize users/groups defined in AD.
  - You can check this by running below:
  ```
  id it1
  groups it1
  hdfs groups it1
  ## groups: it1: no such user
  ```
- Pre-req for below steps: Your AD admin/instructor should have given 'registersssd' user permissions to add the workstation to OU=HadoopNodes (needed to run 'adcli join' successfully)

- *Note: the below is just a sample way of using SSSD.  It will vary completely by environment and needs tuning and testing for your environment.*

- **Run the steps in this section on each node**

```
ad_user="registersssd"
ad_domain="lab.hortonworks.net"
ad_dc="ad01.lab.hortonworks.net"
ad_root="dc=lab,dc=hortonworks,dc=net"
ad_ou="ou=HadoopNodes,${ad_root}"
ad_realm=${ad_domain^^}

sudo kinit ${ad_user}
## enter BadPass#1 for password
```

```
sudo yum makecache fast
##sudo yum -y -q install epel-release ## epel is required for adcli   --Erik Maxwell - epel not required in RHEL 7 for adcli
sudo yum -y -q install sssd oddjob-mkhomedir authconfig sssd-krb5 sssd-ad sssd-tools
sudo yum -y -q install adcli
```

```
#paste all the lines in this block together, in one shot
sudo adcli join -v \
  --domain-controller=${ad_dc} \
  --domain-ou="${ad_ou}" \
  --login-ccache="/tmp/krb5cc_0" \
  --login-user="${ad_user}" \
  -v \
  --show-details
 
## This will output a lot of text. In the middle you should see something like below:  
## ! Couldn't find a computer container in the ou, creating computer account directly in: ou=HadoopNodes,dc=lab,dc=hortonworks,dc=net
## * Calculated computer account: CN=IP-172-30-0-206,ou=HadoopNodes,dc=lab,dc=hortonworks,dc=net
## * Created computer account: CN=IP-172-30-0-206,ou=HadoopNodes,dc=lab,dc=hortonworks,dc=net  
```


```
#paste all the lines in this block together, in one shot
sudo tee /etc/sssd/sssd.conf > /dev/null <<EOF
[sssd]
## master & data nodes only require nss. Edge nodes require pam.
services = nss, pam, ssh, autofs, pac
config_file_version = 2
domains = ${ad_realm}
override_space = _

[domain/${ad_realm}]
id_provider = ad
ad_server = ${ad_dc}
#ad_server = ad01, ad02, ad03
#ad_backup_server = ad-backup01, 02, 03
auth_provider = ad
chpass_provider = ad
access_provider = ad
enumerate = False
krb5_realm = ${ad_realm}
ldap_schema = ad
ldap_id_mapping = True
cache_credentials = True
ldap_access_order = expire
ldap_account_expire_policy = ad
ldap_force_upper_case_realm = true
fallback_homedir = /home/%d/%u
default_shell = /bin/false
ldap_referrals = false

[nss]
memcache_timeout = 3600
override_shell = /bin/bash
EOF
```

```
sudo chmod 0600 /etc/sssd/sssd.conf
sudo service sssd restart
sudo authconfig --enablesssd --enablesssdauth --enablemkhomedir --enablelocauthorize --update

sudo chkconfig oddjobd on
sudo service oddjobd restart
sudo chkconfig sssd on
sudo service sssd restart

sudo kdestroy
```

- Confirm that your nodes OS can now recognize AD users
```
id sales1
groups sales1
```


### Refresh HDFS User-Group mappings

- **Once the above is completed on all nodes you need to refresh the user group mappings in HDFS & YARN by running the below commands**

- **Restart HDFS service via Ambari**. This is needed for Hadoop to recognize the group mappings (else the `hdfs groups` command will not work)

- Execute the following on the Ambari node:
```
export PASSWORD=BadPass#1

#detect name of cluster
output=`curl -k -u hadoopadmin:$PASSWORD -i -H 'X-Requested-By: ambari'  https://localhost:8443/api/v1/clusters`
cluster=`echo $output | sed -n 's/.*"cluster_name" : "\([^\"]*\)".*/\1/p'`

#refresh user and group mappings
sudo sudo -u hdfs kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-"${cluster,,}"
sudo sudo -u hdfs hdfs dfsadmin -refreshUserToGroupsMappings
```

- Execute the following on the node where the YARN ResourceManager is installed:
```
sudo sudo -u yarn kinit -kt /etc/security/keytabs/yarn.service.keytab yarn/$(hostname -f)@LAB.HORTONWORKS.NET
sudo sudo -u yarn yarn rmadmin -refreshUserToGroupsMappings
```


- kinit as an end user (password is BadPass#1)
```
kinit hr1
```

- check the group mappings
```
hdfs groups
yarn rmadmin -getGroups hr1
```

- output should look like below, indicating both OS-level and hadoop-level group mappings :
```
$ hdfs groups
hr1@LAB.HORTONWORKS.NET : domain_users hr hadoop-users
$ yarn rmadmin -getGroups hr1
hr1 : domain_users hr hadoop-users
```

- remove kerberos ticket
```
kdestroy
```

### Test OS/AD integration and Kerberos security

- Login as sales1 user and try to access the same /tmp/hive HDFS dir
```
sudo su - sales1

hdfs dfs -ls /tmp/hive   
## since we did not authenticate, this fails with GSSException: No valid credentials provided

#authenticate
kinit
##enter BadPass#1

klist
## shows the principal for sales1

hdfs dfs -ls /tmp/hive 
## fails with Permission denied

#Now try to get around security by setting the same env variable
export HADOOP_USER_NAME=hdfs
hdfs dfs -ls /tmp/hive 

#log out as sales1
logout
```
- Notice that now that the cluster is kerberized, we were not able to circumvent security by setting the env var 

### Kerberos for Ambari Views

For Ambari Views to access the cluster, Ambari must be configured to use Kerberos to access the cluster. The Kerberos wizard handles this configuration for you (as of Ambari 2.4).

For those configurations to take affect, execute the following on the Ambari Server:

```
sudo ambari-server restart
```

### Enabling SPNEGO Authentication for Hadoop

- Needed to secure the Hadoop components webUIs (e.g. Namenode UI, JobHistory UI, Yarn ResourceManager UI etc...)

- Run steps on ambari server node

- Create Secret Key Used for Signing Authentication Tokens
```
sudo dd if=/dev/urandom of=/etc/security/http_secret bs=1024 count=1
sudo chown hdfs:hadoop /etc/security/http_secret
sudo chmod 440 /etc/security/http_secret
```
- Place the file in Ambari resources dir so it gets pushed to all nodes
```
sudo cp /etc/security/http_secret /var/lib/ambari-server/resources/host_scripts/
sudo ambari-server restart
```

- Wait 30 seconds for the http_secret file to get pushed to all nodes under /var/lib/ambari-agent/cache/host_scripts

- On non-Ambari nodes, once the above file is available, run below to put it in right dir and correct its permissions
```
sudo cp /var/lib/ambari-agent/cache/host_scripts/http_secret /etc/security/
sudo chown hdfs:hadoop /etc/security/http_secret
sudo chmod 440 /etc/security/http_secret
```


- In Ambari > HDFS > Configs, set the below
  - Under Advanced core-site:
    - hadoop.http.authentication.simple.anonymous.allowed=false
  
  - Under Custom core-site, add the below properties (using bulk add tab):
  
  ```
  hadoop.http.authentication.signature.secret.file=/etc/security/http_secret
  hadoop.http.authentication.type=kerberos
  hadoop.http.authentication.kerberos.keytab=/etc/security/keytabs/spnego.service.keytab
  hadoop.http.authentication.kerberos.principal=HTTP/_HOST@LAB.HORTONWORKS.NET
  hadoop.http.authentication.cookie.domain=lab.hortonworks.net
  hadoop.http.filter.initializers=org.apache.hadoop.security.AuthenticationFilterInitializer
  ```
- Save configs

- Restart all services that require restart (HDFS, Mapreduce, YARN, HBase). You can use the 'Actions' > 'Restart All Required' button to restart all the services in one shot


![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Ambari-restart-services.png)

- Now when you try to open any of the web UIs like below you will get `401: Authentication required`
  - HDFS: Namenode UI
  - Mapreduce: Job history UI
  - YARN: Resource Manager UI