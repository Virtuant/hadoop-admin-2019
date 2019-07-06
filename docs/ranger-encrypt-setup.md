## Ranger Encryption setup

- Goal: In this lab we will install Ranger KMS via Ambari. Next we will create some encryption keys and use them to create encryption zones (EZs) and copy files into them. Reference: [docs](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.4/bk_Ranger_KMS_Admin_Guide/content/ch_ranger_kms_overview.html)

- In this section we will have to setup proxyusers. This is done to enable *impersonation* whereby a superuser can submit jobs or access hdfs on behalf of another user (e.g. because superuser has kerberos credentials but user joe doesn’t have any)
  - For more details on this, refer to the [doc](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/Superusers.html)

- Before starting KMS install, find and note down the below piece of information. These will be used during KMS install
  - Find the internal hostname of host running *Mysql* and note it down
    - From Ambari > Hive > Mysql > click the 'Mysql Server' hyperlink. The internal hostname should appear in upper left of the page.

  
- Open Ambari > start 'Add service' wizard > select 'Ranger KMS'.
- Pick any node to install on
- Keep the default configs except for 
  - under Ambari > Ranger KMS > Settings tab :
    - Ranger KMS DB host: <FQDN of Mysql>
    - Ranger KMS DB password: `BadPass#1` 
    - DBA password: `BadPass#1`
    - KMS master secret password: `BadPass#1`
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-KMS-enhancedconfig1.png) 
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-KMS-enhancedconfig2.png) 
    
        
  - Custom kms-site (to avoid adding one at a time, you can use 'bulk add' mode):
      - hadoop.kms.proxyuser.hive.users=*
      - hadoop.kms.proxyuser.oozie.users=*
      - hadoop.kms.proxyuser.HTTP.users=*
      - hadoop.kms.proxyuser.ambari.users=*
      - hadoop.kms.proxyuser.yarn.users=*
      - hadoop.kms.proxyuser.hive.hosts=*
      - hadoop.kms.proxyuser.oozie.hosts=*
      - hadoop.kms.proxyuser.HTTP.hosts=*
      - hadoop.kms.proxyuser.ambari.hosts=*
      - hadoop.kms.proxyuser.yarn.hosts=*    
      - hadoop.kms.proxyuser.keyadmin.groups=*
      - hadoop.kms.proxyuser.keyadmin.hosts=*
      - hadoop.kms.proxyuser.keyadmin.users=*      
        ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-KMS-proxy.png) 

- Click Next > Proceed Anyway to proceed with the wizard

- If prompted, on Configure Identities page, you may have to enter your AD admin credentials:
  - Admin principal: `hadoopadmin@LAB.HORTONWORKS.NET`
  - Admin password: BadPass#1
  - Check the "Save admin credentials" checkbox
  
- Click Next > Deploy to install RangerKMS
        
- Confirm these properties got populated to kms://http@(kmshostname):9292/kms
  - HDFS > Configs > Advanced core-site:
    - hadoop.security.key.provider.path
  - HDFS > Configs > Advanced hdfs-site:
    - dfs.encryption.key.provider.uri  
    
- Restart the services that require it e.g. HDFS, Mapreduce, YARN via Actions > Restart All Required

- Restart Ranger and RangerKMS services.

- (Optional) Add another KMS:
  - Ambari > Ranger KMS > Service Actions > Add Ranger KMS Server > Pick any host
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-add-KMS.png) 
  - After it is installed, you can start it by:
    - Ambari > Ranger KMS > Service Actions > Start
    
  - Once started you will see multiple KMS Servers running in Ambari:  
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-multiple-KMS.png) 

------------------

# Lab 6b

## Ranger KMS/Data encryption exercise

- Before we can start exercising HDFS encryption, we will need to set:
  - policy for hadoopadmin access to HDFS
  - policy for hadoopadmin access to Hive  
  - policy for hadoopadmin access to the KMS keys we created

  - Add the user hadoopadmin to the Ranger HDFS global policies. 
    - Access Manager > HDFS > (clustername)_hdfs   
    - This will open the list of HDFS policies
   ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HDFS-list.png) 
    - Edit the 'all - path' global policy (the first one) and add hadoopadmin to global HDFS policy and Save 
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HDFS-add-hadoopadmin.png) 
    - Your policy now includes hadoopadmin
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HDFS-list-after.png) 
    
  - Add the user hadoopadmin to the Ranger Hive global policies. (Hive has two global policies: one on Hive tables, and one on Hive UDFs)
    - Access Manager > HIVE > (clustername)_hive   
    - This will open the list of HIVE policies
    [Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HIVE-list.png) 
    - Edit the 'all - database, table, column' global policy (the first one) and add hadoopadmin to global HIVE policy and Save  
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HIVE-add-hadoopadmin-table.png) 
    - Edit the 'all - database, udf' global policy (the second one) and add hadoopadmin to global HIVE policy and Save 
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HIVE-add-hadoopadmin-udf.png) 
    - Your policies now includes hadoopadmin
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HIVE-list-after.png) 
     
  - Add policy for keyadmin to be able to access /ranger/audit/kms
    - First Create the hdfs directory for Ranger KMS Audit
    ```
    #run below on Ambari node

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
    
    #Create the Ranger KMS Audit Directory 
    sudo -u hdfs hdfs dfs -mkdir -p /ranger/audit/kms
    sudo -u hdfs hdfs dfs -chown -R kms:hdfs /ranger/audit/kms
    sudo -u hdfs hdfs dfs -chmod 700 /ranger/audit/kms
    sudo -u hdfs hdfs dfs -ls /ranger/audit/kms
    ```
    - Access Manager > HDFS > (clustername)_hdfs   
    - This will open the list of HDFS policies
    - Create a new policy for keyadmin to be able to access /ranger/audit/kms and Save 
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HDFS-keyadmin.png) 
    - Your policy has been added
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-HDFS-keyadmin.png) 
  
  - Give keyadmin permission to view Audits screen in Ranger:
    - Settings tab > Permissions
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-user-permissions.png)
    - Click 'Audit' to change users who have access to Audit screen
    - Under 'Select User', add 'keyadmin' user
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-user-permissions-audits.png)
    - Save
  
    
- Logout of Ranger
  - Top right > admin > Logout      
- Login to Ranger as keyadmin/keyadmin
- Confirm the KMS repo was setup correctly
  - Under Service Manager > KMS > Click the Edit icon (next to the trash icon) to edit the KMS repo
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-edit-repo.png) 
  - Click 'Test connection' and confirm it works

- Create a key called testkey - for reference: see [doc](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.0/bk_security/content/use_ranger_kms.html)
  - Select Encryption > Key Management
  - Select KMS service > pick your kms > Add new Key
    - if an error is thrown, go back and test connection as described in previous step
  - Create a key called `testkey` > Save
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/screenshots/Ranger-KMS-createkey.png)

- Similarly, create another key called `testkey2`
  - Select Encryption > Key Management
  - Select KMS service > pick your kms > Add new Key
  - Create a key called `testkey2` > Save  

- Add user `hadoopadmin` to default KMS key policy
  - Click Access Manager tab
  - Click Service Manager > KMS > (clustername)_kms link
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-KMS-policy.png)

  - Edit the default policy
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-KMS-edit-policy.png)
  
  - Under 'Select User', Add `hadoopadmin` user and click Save
   ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-KMS-policy-add-nn.png)
  
    - Note that:
      - `hdfs` user  needs `GetMetaData` and `GenerateEEK` privilege - HDP 2.5
      - `nn` user  needs `GetMetaData` and `GenerateEEK` privilege - HDP 2.4
      - `hive` user needs `GetMetaData` and `DecryptEEK` privilege

  
- Run below to create a zone using the key and perform basic key and encryption zone (EZ) exercises 
  - Create EZs using keys
  - Copy file to EZs
  - Delete file from EZ
  - View contents for raw file
  - Prevent access to raw file
  - Copy file across EZs
  - move hive warehouse dir to EZ
  
```
#run below on Ambari node

export PASSWORD=BadPass#1

#detect name of cluster
output=`curl -u hadoopadmin:$PASSWORD -k -i -H 'X-Requested-By: ambari'  https://localhost:8443/api/v1/clusters`
cluster=`echo $output | sed -n 's/.*"cluster_name" : "\([^\"]*\)".*/\1/p'`

echo $cluster
## this should show the name of your cluster

## if not you can manully set this as below
## cluster=Security-HWX-LabTesting-XXXX

#first we will run login 3 different users: hdfs, hadoopadmin, sales1

#kinit as hadoopadmin and sales using BadPass#1 
sudo -u hadoopadmin kinit
## enter BadPass#1
sudo -u sales1 kinit
## enter BadPass#1

#then kinit as hdfs using the headless keytab and the principal name
sudo -u hdfs kinit -kt /etc/security/keytabs/hdfs.headless.keytab "hdfs-${cluster,,}"

#as hadoopadmin list the keys and their metadata
sudo -u hadoopadmin hadoop key list -metadata

#as hadoopadmin create dirs for EZs
sudo -u hadoopadmin hdfs dfs -mkdir /zone_encr
sudo -u hadoopadmin hdfs dfs -mkdir /zone_encr2

#as hdfs create 2 EZs using the 2 keys
sudo -u hdfs hdfs crypto -createZone -keyName testkey -path /zone_encr
sudo -u hdfs hdfs crypto -createZone -keyName testkey2 -path /zone_encr2
# if you get 'RemoteException' error it means you have not given namenode user permissions on testkey by creating a policy for KMS in Ranger

#check EZs got created
sudo -u hdfs hdfs crypto -listZones  

#create test files
sudo -u hadoopadmin echo "My test file1" > /tmp/test1.log
sudo -u hadoopadmin echo "My test file2" > /tmp/test2.log

#copy files to EZs
sudo -u hadoopadmin hdfs dfs -copyFromLocal /tmp/test1.log /zone_encr
sudo -u hadoopadmin hdfs dfs -copyFromLocal /tmp/test2.log /zone_encr

sudo -u hadoopadmin hdfs dfs -copyFromLocal /tmp/test2.log /zone_encr2

#Notice that hadoopadmin allowed to decrypt EEK but not sales user (since there is no Ranger policy allowing this)
sudo -u hadoopadmin hdfs dfs -cat /zone_encr/test1.log
sudo -u hadoopadmin hdfs dfs -cat /zone_encr2/test2.log
#this should work

sudo -u sales1      hdfs dfs -cat /zone_encr/test1.log
## this should give you below error
## cat: User:sales1 not allowed to do 'DECRYPT_EEK' on 'testkey'
```

- Check the Ranger > Audit page and notice that the request from hadoopadmin was allowed but the request from sales1 was denied
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-KMS-audit.png)

- Now lets test deleting and copying files between EZs - ([Reference doc](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.4/bk_hdfs_admin_tools/content/copy-to-from-encr-zone.html))
```
#try to remove file from EZ using usual -rm command (note: Trash Support for deletion in HDFS encryption zone has been added since HDP2.4.3)
sudo -u hadoopadmin hdfs dfs -rm /zone_encr/test2.log
## rm: Failed to move to trash.... /zone_encr/test2.log can't be moved from an encryption zone.

#recall that to delete a file from EZ you need to specify the skipTrash option
sudo -u hadoopadmin hdfs dfs -rm -skipTrash /zone_encr/test2.log

#TODO: looks like -skiptrash no loner needed?

#confirm that test2.log was deleted and that zone_encr only contains test1.log
sudo -u hadoopadmin hdfs dfs -ls  /zone_encr/
 
#copy a file between EZs using distcp with -skipcrccheck option
sudo -u hadoopadmin hadoop distcp -skipcrccheck -update /zone_encr2/test2.log /zone_encr/
```
- Lets now look at the contents of the raw file
```
#View contents of raw file in encrypted zone as hdfs super user. This should show some encrypted characters
sudo -u hdfs hdfs dfs -cat /.reserved/raw/zone_encr/test1.log

#Prevent user hdfs from reading the file by setting security.hdfs.unreadable.by.superuser attribute. Note that this attribute can only be set on files and can never be removed.
sudo -u hdfs hdfs dfs -setfattr -n security.hdfs.unreadable.by.superuser  /.reserved/raw/zone_encr/test1.log

# Now as hdfs super user, try to read the files or the contents of the raw file
sudo -u hdfs hdfs dfs -cat /.reserved/raw/zone_encr/test1.log

## You should get below error
##cat: Access is denied for hdfs since the superuser is not allowed to perform this operation.

```

- Configure Hive for HDFS Encryption using testkey. [Reference](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.3.4/bk_hdfs_admin_tools/content/hive-access-encr.html)
```
sudo -u hadoopadmin hdfs dfs -mv /apps/hive /apps/hive-old
sudo -u hadoopadmin hdfs dfs -mkdir /apps/hive
sudo -u hdfs hdfs crypto -createZone -keyName testkey -path /apps/hive
sudo -u hadoopadmin hadoop distcp -skipcrccheck -update /apps/hive-old/warehouse /apps/hive/warehouse
```

- To configure the Hive scratch directory (hive.exec.scratchdir) so that it resides inside the encryption zone:
  - Ambari > Hive > Configs > Advanced 
    - hive.exec.scratchdir = /apps/hive/tmp
  - Restart Hive
  

- Make sure that the permissions for /apps/hive/tmp are set to 1777
```
sudo -u hdfs hdfs dfs -chmod -R 1777 /apps/hive/tmp
```

- Confirm permissions by accessing the scratch dir as sales1
```
sudo -u sales1 hdfs dfs -ls /apps/hive/tmp
## this should provide listing
```

- Destroy ticket for sales1
```
sudo -u sales1 kdestroy
```

- Logout of Ranger as keyadmin user
