## Lab: Using HDFS with Ambari

**Objective**: Begin to get acquainted with Hadoops file system. And manipulate files in HDFS, the Hadoop Distributed File System.

**Exercise directory**: `~/data`

In this section, you will download the sensor data and load that into HDFS using Ambari User Views. You will get introduced to the Ambari Files User View to manage files. You can perform tasks like create directories, navigate file systems and upload files to HDFS.  In addition, you’ll perform a few other file-related tasks as well.  Once you get the basics, you will create two directories and then load two files into HDFS using the Ambari Files User View.

----

### HDFS backdrop

A single physical machine gets saturated with its storage capacity as the data grows. This growth drives the need to partition your data across separate machines. This type of File system that manages storage of data across a network of machines is called Distributed File Systems. HDFS is a core component of Apache Hadoop and is designed to store large files with streaming data access patterns, running on clusters of commodity hardware. With Hortonworks Data Platform (HDP), HDFS is now expanded to support heterogeneous storage media within the HDFS cluster.

#### Download and Extract Sensor Data Files

Download the sample sensor data either local or contained in a compressed (.zip) folder here:  [Geolocation.zip](https://raw.githubusercontent.com/hortonworks/data-tutorials/master/tutorials/hdp/hadoop-tutorial-getting-started-with-hdp/assets/datasets/Geolocation.zip)

Save the Geolocation.zip file to your computer, then extract the files. You should see a Geolocation folder that contains the following files:
    * geolocation.csv –  collected geolocation data from the trucks. It contains records showing truck location, date, time, type of event, speed, etc.
    * trucks.csv – exported from a relational database and it shows information on truck models, driverid, truckid, and aggregated mileage info.

### Load the Sensor Data into HDFS

1. Logon to Ambari

2. Go to Ambari Dashboard and open Files View.

![files-view](https://user-images.githubusercontent.com/558905/54851996-f12a6f80-4cc1-11e9-8d93-9c1dbbc874d1.jpg)

3. Start from the top root of the HDFS file system, you will see all the files the logged in user has access to see:

![root-files-view-800x412](https://user-images.githubusercontent.com/558905/54851998-f12a6f80-4cc1-11e9-8b7a-ee4090916998.jpg)

4. Navigate to `/tmp/` directory by clicking on the directory links.

5. Create directory data. Click the ![new_folder_icon_lab1](https://user-images.githubusercontent.com/558905/54852691-f4bef600-4cc3-11e9-8e92-33a9d4aec0e4.png) button to create that directory. Then navigate to it. The directory path you should see: `/tmp/data`

![add-new-folder-800x74](https://user-images.githubusercontent.com/558905/54851994-f12a6f80-4cc1-11e9-8abc-5c99d01564d5.jpg)

### Upload Geolocation and Trucks CSV Files to data Folder

1. If you’re not already in your newly created directory path `/tmp/data`, go to the data folder. Then click on the 
![upload_icon_lab1](https://user-images.githubusercontent.com/558905/54852693-f8527d00-4cc3-11e9-9cd7-e86c9b65f101.png) button to upload the corresponding geolocation.csv and trucks.csv files into it.

2. An Upload file window will appear, click on the cloud symbol.

![upload_file_lab1-800x252](https://user-images.githubusercontent.com/558905/54852000-f12a6f80-4cc1-11e9-9f99-810f83f05f4f.jpg)

3. Another window will appear, navigate to the destination the two csv files were downloaded. Click on one at a time, press open to complete the upload. Repeat the process until both files are uploaded.

![upload_file_window_lab1](https://user-images.githubusercontent.com/558905/54852001-f1c30600-4cc1-11e9-9409-0b0e4f2ee563.png)

Both files are uploaded to HDFS as shown in the Files View UI:

![uploaded-files-800x393](https://user-images.githubusercontent.com/558905/54851993-f12a6f80-4cc1-11e9-9f32-a5ce5c3f854e.jpg)

You can also perform the following operations on a file or folder by clicking on the entity’s row: `Open, Rename, Permissions, Delete, Copy, Move, Download and Concatenate`.

### Set Write Permissions to Write to data Folder

    1. click on the data folder’s row, which is contained within the directory path `/tmp/`.
    2. Click Permissions.
    3. Make sure that the background of all the write boxes are checked (blue).

![edit-permissions-800x258](https://user-images.githubusercontent.com/558905/54851995-f12a6f80-4cc1-11e9-84af-96b00bf42451.jpg)

### Results

Congratulations! Let’s summarize the skills and knowledge we acquired from this tutorial. We learned Hadoop Distributed File System (HDFS) was built to manage storing data across multiple machines. Now we can upload data into the HDFS using Ambari’s HDFS Files view.
