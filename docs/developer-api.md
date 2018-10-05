# Links to helpful info to update this lab: 

https://community.hortonworks.com/articles/2038/how-to-connect-to-hbase-11-using-java-apis.html

https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.4/bk_command-line-installation/content/ref-2a6efe32-d0e1-4e84-9068-4361b8c36dc8.1.html

https://hortonworks.com/blog/pig-as-hadoop-connector-part-two-hbase-jruby-and-sinatra/

https://community.hortonworks.com/questions/73868/hbase-creating-table-using-java-api-not-compiling.html


## Lab: The Developer API

## Objective:
Use the HBase Java API to programmatically Put, Get, and Scan.

### File locations:

### Successful outcome:
YOU WILL:
In this exercise you will use the HBase Java API to programmatically Put, Get, and Scan. For those who would prefer to use Python with the Thrift interface, rather than Java, please use the slides in Appendix A and follow the instructions for the Appendix A exercise at the end of this document.

### Lab Steps

Complete the setup guide

**Exercise directory**:   `~/class/exercises/`  or use the CalculateAverages project in Eclipse

**HBase tables**:         `movie, user, ratings`

In this exercise you will use the HBase Java API to programmatically Put, Get, and Scan. For those who would prefer to use Python with the Thrift interface, rather than Java, please use the slides in Appendix A and follow the instructions for the Appendix A exercise at the end of this document.

There are three sets of code available to work with: stub, hint, and full solutions. 

- The stub solution has only the minimum classes or files set up. 
- The hint solution has most of the code already in place, leaving you to fill in just a few lines. The hints have TODOs that mark the places where code needs to be filled in. 
- The full solutions implement the complete answer, with comments. Choose the solution that fits your comfort level.

### Integrated Development Environments

There are many Integrated Development Environments (IDEs) available. In this course we have chosen to use Eclipse, a very popular open source IDE. We recommend that you use Eclipse rather than a command-line editor such as vi during the course.

----

### Setup and Start Eclipse

You will use Eclipse in this exercise and future ones. To move the HBase class projects to the Eclipse workspace, run a script to ready the Eclipse environment and the operating system environment.

1.  The Eclipse projects for the exercises have already been installed but should you need to start fresh, issue the following command from a terminal window:

```console
    $ ~/scripts/hbase/hbase_eclipse_projects.sh
```
    
1.  Find the Eclipse icon on your VM’s desktop and double-click the icon to start Eclipse.

### Configuring Eclipse

There is a file synchronization warning displayed by Eclipse. To remove this warning, select your project and press “F5” to refresh the project. You can also use this alternate method if you prefer:

1.  Select Window -> Preferences -> General -> Startup and Shutdown.
2.  On the dialog window shown below, select “Refresh workspace on startup”
3.  Click Apply, then OK
4.  Restart Eclipse for the change to take effect.

### Editing Source Code

Select a Java program from the left window pane and edit it in the right window pane:

### Creating and Using a Jar File

You will run your program from the command line by compiling your program and then creating a jar file. To create a jar file:

1. Right-click on your project name, e.g. CalculateAverages

1. Select _Export -> Java -> Jar File_

1. Click _Next_

1. Enter the directory where you would like the jar file to be created 

1. Enter a jar file name

1. Click _Next_, _Next_, and _Finish_

> Note: If no directory is entered, the jar file will be created in the ~/_workspace_ directory.

In your terminal window, navigate to the directory where your newly created jar file is stored and issue the following command to run your program now – be sure to include the package name if necessary, e.g. ,solution, hints, stub.

```console
    $ java -cp `hbase classpath`:_myJar.jar solution|hints|stubs_._myClass_
```
    
### Scan Tables with the Java API

### Java API

The CalculateAverages project has been configured with all necessary Jar files included on the classpath. The various solutions are in different packages in the project. Choose the one you are most comfortable with to start development.

> NOTE: If you decide not to use Eclipse, you can find the files for all of the exercises are in: `~/class/exercises/`

An HBase best practice is to avoid repeated hard coding of table and column descriptor names. The Java code contains a `Common.java` file that contains all of the names and byte arrays in one place. The code for that project will reference the `Common` class for any names or byte arrays.

### Calculate Average Ratings

The user table has a column family called ratings. Within this column family, column descriptors correspond to the row keys of the movie table, and the cell values are the ratings given to the movies by that user. For example, if a row has a column named 3001, with a value of 4, the user gave the movie whose row key in the movie table is 3001, a rating of 4.

1.  Use a scan to calculate the average rating for a movie by retrieving all ratings for that movie given by all users.

1.  Add the average to the movie’s row in the movie table as a new column named `average`, and add a column named `count` to hold the total number of ratings given to the movie. Store the values as UTF strings, not numbers.

1.  Use the HBase shell to verify that your program works correctly by inspecting the movie.

1.  You can reduce the amount of output shown by using the COLUMNS directive to specify the column family and column qualifiers of interest. In this case, you are interested in the newly added average and count column qualifiers.

    ```console
    hbase> scan 'movie',{COLUMNS => ['info:average',  'info:count']}
    ```
    
### Retrieve Movies

1.  Write a separate program to output the data in the row for each of the following row keys, rounded to two decimal places.

    ```console
    '45'
    '67'
    '78'
    '190'
    '2001'
    ```
    
1.  To verify that your program worked correctly, issue the following command in the HBase shell.

    ```console
    hbase> get 'movie', '67'
    ```
    
1.  Now view the ratings that contributed to the average rating for the movie with movie ID = '67'.

    ```console
    hbase> scan 'user',{COLUMNS => 'ratings:67'}
    ```
    
### Results:

|ROW|COLUMN+CELL|
|---|---|
|195|column=ratings:67, timestamp=1425403168664, value=4|
|3705|column=ratings:67, timestamp=1425403187282, value=3|
|4277|column=ratings:67, timestamp=1425403197320, value=4|
|5334|column=ratings:67, timestamp=1425403204981, value=2|

 4 row(s) in 0.0820 seconds
