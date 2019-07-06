## Security options for Ambari


### Ambari server as non-root

- Create a user for the Ambari Server if it does not exists
```
useradd -d /var/lib/ambari-server -G hadoop -M -r -s /sbin/nologin ambari
```
- Otherwise - Update the Ambari Server with the following
```
usermod -d /var/lib/ambari-server -G hadoop -s /sbin/nologin ambari
```

- Grant the user 'sudoers' rights. This is required for Ambari Server to create it's Kerberos keytabs. You can remove this after kerberizing the cluster
```
echo 'ambari ALL=(ALL) NOPASSWD:SETENV: /bin/mkdir, /bin/cp, /bin/chmod, /bin/rm, /bin/chown' > /etc/sudoers.d/ambari-server
```

- Also confirm your sudoers file has correct defaults, as per https://docs.hortonworks.com/HDPDocuments/Ambari-2.5.0.3/bk_ambari-security/content/sudo_defaults_server.html

- To setup Ambari server as non-root run below on Ambari-server node:
```
sudo ambari-server setup
```
- Then enter the below at the prompts:
  - OK to continue? y
  - Customize user account for ambari-server daemon? y
  - Enter user account for ambari-server daemon (root):ambari
  - Do you want to change Oracle JDK [y/n] (n)? n
  - Enter advanced database configuration [y/n] (n)? n

- Sample output:
```
$ sudo ambari-server setup
Using python  /usr/bin/python2
Setup ambari-server
Checking SELinux...
SELinux status is 'enabled'
SELinux mode is 'permissive'
WARNING: SELinux is set to 'permissive' mode and temporarily disabled.
OK to continue [y/n] (y)? y
Customize user account for ambari-server daemon [y/n] (n)? y
Enter user account for ambari-server daemon (root):ambari
Adjusting ambari-server permissions and ownership...
Checking firewall status...
Redirecting to /bin/systemctl status  iptables.service

Checking JDK...
Do you want to change Oracle JDK [y/n] (n)? n
Completing setup...
Configuring database...
Enter advanced database configuration [y/n] (n)? n
Configuring database...
Default properties detected. Using built-in database.
Configuring ambari database...
Checking PostgreSQL...
Configuring local database...
Connecting to local database...done.
Configuring PostgreSQL...
Backup for pg_hba found, reconfiguration not required
Extracting system views...
.......
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup' completed successfully.
```
<!---
- Create proxy user settings for ambari user to enable it to become a super user on all hosts (more details on this later):
  - Ambari > HDFS > Configs > Advanced > Custom core-site > Add property > Bulk mode:
```
hadoop.proxyuser.ambari-server.groups=*
hadoop.proxyuser.ambari-server.hosts=* 
```
![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-proxyuser.png)

- Save and restart HDFS
  - Ambari will show that other components need restarting too but you can proceed without restarting those for now to save time (we will restart those later)

--->

### Run ambari-agent as non-root

- For now we will skip configuring Ambari Agents for Non-Root

### Ambari Encrypt Database and LDAP Passwords

- Needed to allow Ambari to cache the admin password. Run below on Ambari-server node:

- To encrypt password, run below
```
sudo ambari-server stop
sudo ambari-server setup-security
```
- Then enter the below at the prompts:
  - enter choice: 2
  - provide master key: BadPass#1
  - re-enter master key: BadPass#1
  - do you want to persist? y

- Then start ambari
```
sudo ambari-server start
```  
- Sample output
```
$ sudo ambari-server setup-security
Using python  /usr/bin/python2
Security setup options...
===========================================================================
Choose one of the following options:
  [1] Enable HTTPS for Ambari server.
  [2] Encrypt passwords stored in ambari.properties file.
  [3] Setup Ambari kerberos JAAS configuration.
  [4] Setup truststore.
  [5] Import certificate to truststore.
===========================================================================
Enter choice, (1-5): 2
Please provide master key for locking the credential store:
Re-enter master key:
Do you want to persist master key. If you choose not to persist, you need to provide the Master Key while starting the ambari server as an env variable named AMBARI_SECURITY_MASTER_KEY or the start will prompt for the master key. Persist [y/n] (y)? y
Adjusting ambari-server permissions and ownership...
Ambari Server 'setup-security' completed successfully.
```

### SSL For Ambari server

- Enables Ambari WebUI to run on HTTPS instead of HTTP

#### Create self-signed certificate

- For this lab we will be generating a self-signed certificate. In production environments you would want to use a signed certificate (either from a public authority or your own CA).

- Generate the certificate & key
```
openssl req -x509 -newkey rsa:4096 -keyout ambari.key -out ambari.crt -days 1000 -nodes -subj "/CN=$(curl icanhazptr.com)"
```

- Move & secure the certificate & key
```
  chown ambari ambari.crt ambari.key
  chmod 0400 ambari.crt ambari.key
  mv ambari.crt /etc/pki/tls/certs/
  mv ambari.key /etc/pki/tls/private/
```

#### Configure Ambari Server for HTTPS (using the above certificate & key)

- Stop Ambari server
```
sudo ambari-server stop
```

- Setup HTTPS for Ambari 
```
# sudo ambari-server setup-security
Using python  /usr/bin/python2
Security setup options...
===========================================================================
Choose one of the following options:
  [1] Enable HTTPS for Ambari server.
  [2] Encrypt passwords stored in ambari.properties file.
  [3] Setup Ambari kerberos JAAS configuration.
  [4] Setup truststore.
  [5] Import certificate to truststore.
===========================================================================
Enter choice, (1-5): 1
Do you want to configure HTTPS [y/n] (y)? y
SSL port [8443] ? 8443
Enter path to Certificate: /etc/pki/tls/certs/ambari.crt
Enter path to Private Key: /etc/pki/tls/private/ambari.key
Please enter password for Private Key: BadPass#1
Importing and saving Certificate...done.
Adjusting ambari-server permissions and ownership...
```

- Start Ambari
```
sudo ambari-server start
```

- Now you can access Ambari on **HTTPS** on port 8443 e.g. https://ec2-52-32-113-77.us-west-2.compute.amazonaws.com:8443
  - If you were not able to access the Ambari UI, make sure you are trying to access *https* not *http*

- Note that the browser will not trust the new self signed ambari certificate. You will need to trust that cert first.
  - If Firefox, you can do this by clicking on 'i understand the risk' > 'Add Exception...'
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/firefox-untrusted.png)  
  - If Chome, you can do this by clicking on 'Advanced' > 'Proceed to xxxxxx'
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/chrome-untrusted.png)  


### Setup Ambari/AD sync

Run below on only Ambari node:

- This puts our AD-specific settings into variables for use in the following command
```
ad_host="ad01.lab.hortonworks.net"
ad_root="ou=CorpUsers,dc=lab,dc=hortonworks,dc=net"
ad_user="cn=ldap-reader,ou=ServiceUsers,dc=lab,dc=hortonworks,dc=net"
```

- Execute the following to configure Ambari to sync with LDAP.
- Use the default password used throughout this course.
  ```
  sudo ambari-server setup-ldap \
    --ldap-url=${ad_host}:389 \
    --ldap-secondary-url= \
    --ldap-ssl=false \
    --ldap-base-dn=${ad_root} \
    --ldap-manager-dn=${ad_user} \
    --ldap-bind-anonym=false \
    --ldap-dn=distinguishedName \
    --ldap-member-attr=member \
    --ldap-group-attr=cn \
    --ldap-group-class=group \
    --ldap-user-class=user \
    --ldap-user-attr=sAMAccountName \
    --ldap-save-settings \
    --ldap-bind-anonym=false \
    --ldap-referral=
  ```
   ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-setup-LDAP-new.png)

- Restart Ambari server
  ```
   sudo ambari-server restart
  ```

- Run LDAPsync to sync only the groups we want
  - When prompted for user/password, use the *local* Ambari admin credentials (i.e. admin/BadPass#1)
  ```
  echo hadoop-users,hr,sales,legal,hadoop-admins > groups.txt
  sudo ambari-server sync-ldap --groups groups.txt
  ```
  
  - This should show a summary of what objects were created
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-run-LDAPsync.png)
  
- Give 'hadoop-admin' admin permissions in Ambari to allow the user to manage the cluster
  - Login to Ambari as your local 'admin' user (i.e. admin/BadPass#1)
  - Grant 'hadoopadmin' user permissions to manage the cluster:
    - Click the dropdown on top right of Ambari UI
    - Click 'Manage Ambari'
    - Under 'Users', select 'hadoopadmin'
    - Change 'Ambari Admin' to Yes 
    ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ambari-make-user-admin.png)    
    
- Sign out and then log back into Ambari, this time as 'hadoopadmin' and verify the user has rights to monitor/manage the cluster

- (optional) Disable local 'admin' user using the same 'Manage Ambari' menu

### Ambari views 

Ambari views setup on secure cluster will be covered in later lab so we will skip this for now ([here](https://github.com/HortonworksUniversity/Security_Labs#other-security-features-for-ambari))
