## Ambari views

- Goal: In this lab we will setup Ambari views on kerborized cluster. 

- Change transport mode back to binary in Hive settings:
  - In Ambari, under Hive > Configs > set the below and restart Hive component.
    - hive.server2.transport.mode = binary

- You may also need to change proxy user settings to be less restrictive

- Option 1: Manual setup following [doc](http://docs.hortonworks.com/HDPDocuments/Ambari-2.2.1.0/bk_ambari_views_guide/content/ch_using_ambari_views.html)
 
- Restart HDFS and YARN via Ambari

- Access the views:
  - Files view
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Files-view.png)
  - Hive view
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Hive-view.png)
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Hive-view-viz.png)
  - Pig view
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Pig-view.png)
  - Tez view
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Tez-view.png)  
  ![Image](https://raw.githubusercontent.com/HortonworksUniversity/Security_Labs/master/screenshots/Tez-view-viz.png)
    

###### Enable users to log into Ambari views

- In Ambari follow steps below:
  - On top right of page, click "Manage Ambari"
    - under 'Views': Navigate to Hive > Hive > Under 'Permissions' grant sales1 access to Hive view
    - similarly you can give sales1 access to Files view   
    - similarly you can give others users access to various views

- At this point, you should be able to login to Ambari as sales1 user and navigate to the views

- Test access as different users (hadoopadmin, sales1, hr1 etc). You can create Ranger policies as needed to grant access to particular groups to  particular resources
