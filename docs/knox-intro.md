## Knox 

Goal: In this lab we will configure Apache Knox for AD authentication and make WebHDFS, Hive requests over Knox (after setting the appropriate Ranger authorization polices for access)

### Knox Configuration 

#### Knox Configuration for AD authentication
 
- Run these steps on the node where Knox was installed earlier

- To configure Knox for AD authentication we need to enter AD related properties in topology xml via Ambari

- The problem is it requires us to enter LDAP bind password, but we do not want it exposed as plain text in the Ambari configs

- The solution? Create keystore alias for the ldap manager user (which you will later pass in to the topology via the 'systemUsername' property)
   - Read password for use in following command (this will prompt you for a password and save it in knoxpass environment variable). Enter BadPass#1:
   ```
   read -s -p "Password: " knoxpass
   ```
  - This is a handy way to set an env var without storing the command in your history

   - Create password alias for Knox called knoxLdapSystemPassword
   ```
   sudo -u knox /usr/hdp/current/knox-server/bin/knoxcli.sh create-alias knoxLdapSystemPassword --cluster default --value ${knoxpass}
   unset knoxpass
   ```
  
- Now lets configure Knox to use our AD for authentication. Replace below content in Ambari > Knox > Config > Advanced topology. 
  - How to tell what configs were changed from defaults? 
    - Default configs remain indented below
    - Configurations that were added/modified are not indented
```
        <topology>

            <gateway>

                <provider>
                    <role>authentication</role>
                    <name>ShiroProvider</name>
                    <enabled>true</enabled>
                    <param>
                        <name>sessionTimeout</name>
                        <value>30</value>
                    </param>
                    <param>
                        <name>main.ldapRealm</name>
                        <value>org.apache.hadoop.gateway.shirorealm.KnoxLdapRealm</value> 
                    </param>

<!-- changes for AD/user sync -->

<param>
    <name>main.ldapContextFactory</name>
    <value>org.apache.hadoop.gateway.shirorealm.KnoxLdapContextFactory</value>
</param>

<!-- main.ldapRealm.contextFactory needs to be placed before other main.ldapRealm.contextFactory* entries  -->
<param>
    <name>main.ldapRealm.contextFactory</name>
    <value>$ldapContextFactory</value>
</param>

<!-- AD url -->
<param>
    <name>main.ldapRealm.contextFactory.url</name>
    <value>ldap://ad01.lab.hortonworks.net:389</value> 
</param>

<!-- system user -->
<param>
    <name>main.ldapRealm.contextFactory.systemUsername</name>
    <value>cn=ldap-reader,ou=ServiceUsers,dc=lab,dc=hortonworks,dc=net</value>
</param>

<!-- pass in the password using the alias created earlier -->
<param>
    <name>main.ldapRealm.contextFactory.systemPassword</name>
    <value>${ALIAS=knoxLdapSystemPassword}</value>
</param>

                    <param>
                        <name>main.ldapRealm.contextFactory.authenticationMechanism</name>
                        <value>simple</value>
                    </param>
                    <param>
                        <name>urls./**</name>
                        <value>authcBasic</value> 
                    </param>

<!--  AD groups of users to allow -->
<param>
    <name>main.ldapRealm.searchBase</name>
    <value>ou=CorpUsers,dc=lab,dc=hortonworks,dc=net</value>
</param>
<param>
    <name>main.ldapRealm.userObjectClass</name>
    <value>person</value>
</param>
<param>
    <name>main.ldapRealm.userSearchAttributeName</name>
    <value>sAMAccountName</value>
</param>

<!-- changes needed for group sync-->
<param>
    <name>main.ldapRealm.authorizationEnabled</name>
    <value>true</value>
</param>
<param>
    <name>main.ldapRealm.groupSearchBase</name>
    <value>ou=CorpUsers,dc=lab,dc=hortonworks,dc=net</value>
</param>
<param>
    <name>main.ldapRealm.groupObjectClass</name>
    <value>group</value>
</param>
<param>
    <name>main.ldapRealm.groupIdAttribute</name>
    <value>cn</value>
</param>


                </provider>

                <provider>
                    <role>identity-assertion</role>
                    <name>Default</name>
                    <enabled>true</enabled>
                </provider>

                <provider>
                    <role>authorization</role>
                    <name>XASecurePDPKnox</name>
                    <enabled>true</enabled>
                </provider>

<!--
  Knox HaProvider for Hadoop services
  -->
<provider>
     <role>ha</role>
     <name>HaProvider</name>
     <enabled>true</enabled>
     <param>
         <name>OOZIE</name>
         <value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true</value>
     </param>
     <param>
         <name>HBASE</name>
         <value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true</value>
     </param>
     <param>
         <name>WEBHCAT</name>
         <value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true</value>
     </param>
     <param>
         <name>WEBHDFS</name>
         <value>maxFailoverAttempts=3;failoverSleep=1000;maxRetryAttempts=300;retrySleep=1000;enabled=true</value>
     </param>
     <param>
        <name>HIVE</name>
        <value>maxFailoverAttempts=3;failoverSleep=1000;enabled=true;zookeeperEnsemble=machine1:2181,machine2:2181,machine3:2181;
       zookeeperNamespace=hiveserver2</value>
     </param>
</provider>
<!--
  END Knox HaProvider for Hadoop services
  -->


            </gateway>

            <service>
                <role>NAMENODE</role>
                <url>hdfs://{{namenode_host}}:{{namenode_rpc_port}}</url>
            </service>

            <service>
                <role>JOBTRACKER</role>
                <url>rpc://{{rm_host}}:{{jt_rpc_port}}</url>
            </service>

            <service>
                <role>WEBHDFS</role>
                <url>http://{{namenode_host}}:{{namenode_http_port}}/webhdfs</url>
            </service>

            <service>
                <role>WEBHCAT</role>
                <url>http://{{webhcat_server_host}}:{{templeton_port}}/templeton</url>
            </service>

            <service>
                <role>OOZIE</role>
                <url>http://{{oozie_server_host}}:{{oozie_server_port}}/oozie</url>
            </service>

            <service>
                <role>WEBHBASE</role>
                <url>http://{{hbase_master_host}}:{{hbase_master_port}}</url>
            </service>

            <service>
                <role>HIVE</role>
                <url>http://{{hive_server_host}}:{{hive_http_port}}/{{hive_http_path}}</url>
            </service>

            <service>
                <role>RESOURCEMANAGER</role>
                <url>http://{{rm_host}}:{{rm_port}}/ws</url>
            </service>
        </topology>
```

- Then restart Knox via Ambari

#### HDFS Configuration for Knox

-  Tell Hadoop to allow our users to access Knox from any node of the cluster. Modify the below properties under Ambari > HDFS > Config > Custom core-site  ('users' group should already part of the groups so just add the rest)
  - hadoop.proxyuser.knox.groups=users,hadoop-admins,sales,hr,legal
  - hadoop.proxyuser.knox.hosts=*
    - (better would be to put a comma separated list of the FQDNs of the hosts)
  - Now restart HDFS
  - Without this step you will see an error like below when you run the WebHDFS request later on:
  ```
   org.apache.hadoop.security.authorize.AuthorizationException: User: knox is not allowed to impersonate sales1"
  ```


#### Ranger Configuration for WebHDFS over Knox
  
- Setup a Knox policy for sales group for WEBHDFS by:
- Login to Ranger > Access Manager > KNOX > click the cluster name link > Add new policy
  - Policy name: webhdfs
  - Topology name: default
  - Service name: WEBHDFS
  - Group permissions: sales 
  - Permission: check Allow
  - Add

  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-knox-webhdfs-policy.png)

#### WebHDFS over Knox exercises 

- Now we can post some requests to WebHDFS over Knox to check its working. We will use curl with following arguments:
  - -i (aka –include): used to output HTTP response header information. This will be important when the content of the HTTP Location header is required for subsequent requests.
  - -k (aka –insecure) is used to avoid any issues resulting from the use of demonstration SSL certificates.
  - -u (aka –user) is used to provide the credentials to be used when the client is challenged by the gateway.
  - Note that most of the samples do not use the cookie features of cURL for the sake of simplicity. Therefore we will pass in user credentials with each curl request to authenticate.

- *From the host where Knox is running*, send the below curl request to 8443 port where Knox is running to run `ls` command on `/` dir in HDFS:
```
curl -ik -u sales1:BadPass#1 https://localhost:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS
```
  - This should return json object containing list of dirs/files located in root dir and their attributes

- To avoid passing password on command prompt you can pass in just the username (to avoid having the password captured in the shell history). In this case, you will be prompted for the password  
```
curl -ik -u sales1 https://localhost:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS

## enter BadPass#1
```

- For the remaining examples below, for simplicity, we are passing in the password on the command line, but feel free to remove the password and enter it in manually when prompted

- Try the same request as hr1 and notice it fails with `Error 403 Forbidden` :
  - This is expected since in the policy above, we only allowed sales group to access WebHDFS over Knox
```
curl -ik -u hr1:BadPass#1 https://localhost:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS
```

- Notice that to make the requests over Knox, a kerberos ticket is not needed - the user authenticates by passing in AD/LDAP credentials

- Check in Ranger Audits to confirm the requests were audited:
  - Ranger > Audit > Service type: KNOX

  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-knox-webhdfs-audit.png)


- Other things to access WebHDFS with Knox

  - A. Use cookie to make request without passing in credentials
    - When you ran the previous curl request it would have listed HTTP headers as part of output. One of the headers will be 'Set Cookie'
    - e.g. `Set-Cookie: JSESSIONID=xxxxxxxxxxxxxxx;Path=/gateway/default;Secure;HttpOnly`
    - You can pass in the value from your setup and make the request without passing in credentials:
      - Make sure you copy the JSESSIONID from a request that worked (i.e the one from sales1 not hr1)
  ```
  curl -ik --cookie "JSESSIONID=xxxxxxxxxxxxxxx;Path=/gateway/default;Secure;HttpOnly" -X GET https://localhost:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS
  ```
  
  - B. Open file via WebHDFS
    - Sample command to list files under /tmp:
    ```
    curl -ik -u sales1:BadPass#1 https://localhost:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS
    ```
      - You can run below command to create a test file into /tmp
      
      ```
      echo "Test file" > /tmp/testfile.txt
      sudo -u sales1 kinit
      ## enter BadPass#1
      sudo -u sales1 hdfs dfs -put /tmp/testfile.txt /tmp
      sudo -u sales1 kdestroy
      ```
      
    - Open this file via WebHDFS 
    ```
    curl -ik -u sales1:BadPass#1 -X GET https://localhost:8443/gateway/default/webhdfs/v1/tmp/testfile.txt?op=OPEN
    ```
      - Look at value of Location header. This will contain a long url 
      ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/knox-location.png)
            
    - Access contents of file /tmp/testfile.txt by passing the value from the above Location header
    ```
    curl -ik -u sales1:BadPass#1 -X GET '{https://localhost:8443/gateway/default/webhdfs/data/v1/webhdfs/v1/tmp/testfile.txt?_=AAAACAAAABAAAAEwvyZNDLGGNwahMYZKvaHHaxymBy1YEoe4UCQOqLC7o8fg0z6845kTvMQN_uULGUYGoINYhH5qafY_HjozUseNfkxyrEo313-Fwq8ISt6MKEvLqas1VEwC07-ihmK65Uac8wT-Cmj2BDab5b7EZx9QXv29BONUuzStCGzBYCqD_OIgesHLkhAM6VNOlkgpumr6EBTuTnPTt2mYN6YqBSTX6cc6OhX73WWE6atHy-lv7aSCJ2I98z2btp8XLWWHQDmwKWSmEvtQW6Aj-JGInJQzoDAMnU2eNosdcXaiYH856zC16IfEucdb7SA_mqAymZuhm8lUCvL25hd-bd8p6mn1AZlOn92VySGp2TaaVYGwX-6L9by73bC6sIdi9iKPl3Iv13GEQZEKsTm1a96Bh6ilScmrctk3zmY4vBYp2SjHG9JRJvQgr2XzgA}'
    ```
      
  - C. Use groovy scripts to access WebHDFS
    - Edit the groovy script to set:
      - gateway = "https://localhost:8443/gateway/default"
    ```
    sudo vi /usr/hdp/current/knox-server/samples/ExampleWebHdfsLs.groovy
    ```
    - Run the script and enter credentials when prompted username: sales1 and password: BadPass#1
    ```
    sudo java -jar /usr/hdp/current/knox-server/bin/shell.jar /usr/hdp/current/knox-server/samples/ExampleWebHdfsLs.groovy
    ```
    - Notice output show list of dirs in HDFS
    ```
    [app-logs, apps, ats, hdp, mapred, mr-history, ranger, tmp, user, zone_encr]
    ```
    
  - D. Access via browser 
    - Take the same url we have been hitting via curl and replace localhost with public IP of Knox node (remember to use https!) e.g. **https**://PUBLIC_IP_OF_KNOX_HOST:8443/gateway/default/webhdfs/v1?op=LISTSTATUS
    - Open the URL via browser
    - Login as sales1/BadPass#1
    
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/knox-webhdfs-browser1.png)
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/knox-webhdfs-browser2.png)
     ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/knox-webhdfs-browser3.png)
      

- We have shown how you can use Knox to avoid the end user from having to know about internal details of cluster
  - whether its kerberized or not
  - what the cluster topology is (e.g. what node WebHDFS was running)


#### Hive over Knox 

##### Configure Hive for Knox

- In Ambari, under Hive > Configs > set the below and restart Hive component.
  - hive.server2.transport.mode = http
- Give users access to jks file.
  - This is only for testing since we are using a self-signed cert.
  - This only exposes the truststore, not the keys.
```
sudo chmod o+x /usr/hdp/current/knox-server /usr/hdp/current/knox-server/data /usr/hdp/current/knox-server/data/security /usr/hdp/current/knox-server/data/security/keystores
sudo chmod o+r /usr/hdp/current/knox-server/data/security/keystores/gateway.jks
```

##### Ranger Configuration for Hive over Knox
  
- Setup a Knox policy for sales group for HIVE by:
- Login to Ranger > Access Manager > KNOX > click the cluster name link > Add new policy
  - Policy name: hive
  - Topology name: default
  - Service name: HIVE
  - Group permissions: sales 
  - Permission: check Allow
  - Add

  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-knox-hive-policy.png)


##### Use Hive for Knox

- By default Knox will use a self-signed (untrusted) certificate. To trust the certificate:
  
    - First on Knox node, create the /tmp/knox.crt certificate

```
knoxserver=$(hostname -f)
openssl s_client -connect ${knoxserver}:8443 <<<'' | openssl x509 -out /tmp/knox.crt
```
  - On node where beeline will be run from (e.g. Hive node):
      - copy over the /tmp/knox.crt
        - easiest option is to just open it in `vi` and copy/paste the contents over:
        `vi /tmp/knox.crt`
      - trust the certificate by running the command below      

```
sudo keytool -import -trustcacerts -keystore /etc/pki/java/cacerts -storepass changeit -noprompt -alias knox -file /tmp/knox.crt
```

  - Now connect via beeline, making sure to replace KnoxserverInternalHostName first below:
  
```
beeline -u "jdbc:hive2://KnoxserverInternalHostName:8443/;ssl=true;transportMode=http;httpPath=gateway/default/hive" -n sales1 -p BadPass#1
```

- Notice that in the JDBC connect string for connecting to an secured Hive running in http transport mode:
  - *port changes to Knox's port 8443*
  - *traffic between client and Knox is over HTTPS*
  - *a kerberos principal not longer needs to be passed in*


- Test these users:
  - sales1/BadPass#1 should work
  - hr1/BadPass#1 should *not* work
    - Will fail with:
    ```
    Could not create http connection to jdbc:hive2://hostname:8443/;ssl=true;transportMode=http;httpPath=gateway/default/hive. HTTP Response code: 403 (state=08S01,code=0)
    ```

- Check in Ranger Audits to confirm the requests were audited:
  - Ranger > Audit > Service type: KNOX
  ![Image](https://raw.githubusercontent.com/Virtuant/hadoop-admin-2019/master/screenshots/Ranger-audit-KNOX-hive-summary.png)


- This shows how Knox helps end users access Hive securely over HTTPS using Ranger to set authorization policies and for audits
