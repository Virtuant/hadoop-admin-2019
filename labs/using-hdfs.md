# Possible replacements labs

https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/

https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/1/

https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/2/

https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/3/

## Objective:
begin to get acquainted with Hadoops file system.

### File locations:

### Successful outcome:
You will:
Manipulate files in HDFS, the Hadoop Distributed File System.

### Lab Steps

Complete the setup guide

## Lab: Using HDFS

**Exercise directory**: `~/data`

**Data directory**: `~/data`

**HDFS paths:** `/user/student`

In this exercise you will begin to get acquainted with Hadoop. You will manipulate files in HDFS, the Hadoop Distributed File System.

----

# Links to replace this lab are:

https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/1/
https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/2/
https://hortonworks.com/tutorial/manage-files-on-hdfs-via-cli-ambari-files-view/section/3/

### Hadoop

Hadoop is already installed, configured, and running on your virtual machine. Hadoop is installed in the /usr/lib/hadoop directory.

Most of your interaction with the system will be through commands supported by the bin/hdfs script. If you start a terminal and run this program with no arguments, it prints a help message. To try this, run the following command:

   ```console
   # hdfs
   ```

   > Note: although your command prompt is more verbose, we use “$” to indicate the command prompt for brevity.

### Exploring HDFS

1. Open a terminal window (if one is not already open) by double-clicking the Terminal icon on the desktop.

1. Make sure  the user is added:

   ```console
   # hdfs dfs -ls
   ```
   
   If it is not there, then add the user:
   
      ```console
   # hdfs dfs -mkdir /user/student
   ```

1. Now enter:

   ```console
   # hdfs dfs -ls /
   ```
   
   This shows you the contents of the root directory in HDFS. There will be multiple entries, one of which is /user. Individual users have a “home” directory under this directory, named after their username – your home directory is
   
   ```console
   /user/student
   ```
   
1. View the contents of the /user directory by entering the following:

   ```console
   # hdfs dfs -ls -R /user
   ```
   
   You will see your home directory in the directory listing. The -R is the extended listing.

1. Enter the following:

   ```console
   # hdfs dfs -ls /user/student
   ```
   
   There may be no files, so the command silently exits. This is different than if you ran `hdfs dfs -ls /foo`, which refers to a directory that does not exist and which would display an error message.

   > Note: that the directory structure in HDFS has nothing to do with the directory structure of the local filesystem; they are completely separate namespaces.

### Uploading Files

Besides browsing the existing filesystem, another important thing you can do with FsShell is to upload new data into HDFS.

1. Change directories to the directory containing the sample data you will be using in the course.

   ```console
   # cd ~/data
   ```
   
   If you perform a “regular” ls command in this directory, you will see a few files, including one named `shakespeare.tar.gz`.

1. Unzip the tar.gz by running:

   ```console
   # tar -zxvf shakespeare.tar.gz
   ```
   
   This creates a directory named `shakespeare/` containing several files on your local filesystem. If you have used this VM for a previous course, we recommend that you unzip the tar file again to ensure you have the file required for this lab.

1. Change to the shakespeare directory and view the files:

   ```console
   # cd shakespeare
   # ls
   ```
   
   You will see a few files, one of which is named glossary.

1. Copy this file into HDFS:

   ```console
   # hdfs dfs -put glossary /user/student/glossary
   ```
   
   This copies the local glossary file into the remote HDFS directory named
   
   ```console
   /user/student/
   ```

1. List the contents of your HDFS home directory now:

   ```console
   # hdfs dfs -ls /user/student
   ```
   
   You should see an entry for the glossary file.

1. Now try the same fs -ls command but without a path argument:

   ```console
   # hdfs dfs -ls
   ```
   
   You should see the same results. If you do not pass a directory name to the -ls command, it assumes you mean your home directory.

1. Enter:

   ```console
   # hdfs dfs -tail glossary
   ```
   
   This prints the last 1KB of the glossary to your terminal. This command is handy for viewing the output of MapReduce programs. Very often, an individual output file of a MapReduce program is very large, making it inconvenient to view the entire file in the terminal.

### Other Commands

There are several other commands associated with the FsShell subsystem which let you perform most common filesystem manipulations: `rm, rm -r (recursive rm), mv, cp, mkdir`, etc.

1. In the terminal window, enter:

   ```console
   # hdfs dfs
   ```
   You see a help message describing all the commands associated with this subsystem.

1. In the terminal window, enter:

   ```console
   # hdfs dfs -help
   ```
   This will give a more complete description of each FsShell command and the arguments for it.

1. Enter:

   ```console
   # hdfs dfs -cat glossary | tail -n 50
   ```

This prints the last 50 lines of the glossary to your terminal. This command is handy for viewing the output of MapReduce programs. Very often, an individual output file of a MapReduce program is very large, making it inconvenient to view the entire file in the terminal. For this reason, it’s often a good idea to pipe the output of the fs -cat command into head, tail, more, or less.
