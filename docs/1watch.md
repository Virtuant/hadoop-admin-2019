https://github.com/hortonworks/data-tutorials/tree/master/tutorials/hdf
https://developer.lightbend.com/docs/alpakka/current/index.html
https://www.onica.com/blog/apache-zeppelin-with-phoenix-and-hbase-interpreters-on-amazon-emr/
https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration
https://goranzugic.wordpress.com/2016/04/11/hbase-schema/
https://www.educba.com/hbase-vs-cassandra/
https://www.hindawi.com/journals/cmmm/2017/6120820/
https://github.com/IBM/sparksql-for-hbase


## Setup Ambari auto restart:

SupportKB

Create ambari-server.service file then run systmectl enable.

1. touch /etc/systemd/system/ambari-server.service 
2. chmod 664 /etc/systemd/system/ambari-server.service 

3. Add the below line to ambari-server.service file 
[Unit] 
Description=ambari-server 
After=network.target 

[Service] 
ExecStart=/usr/sbin/ambari-server start 
Type=forking 

[Install] 
WantedBy=default.target 

4. systemctl enable ambari-server 
5. systemctl daemon-reload 
6. systemctl list-unit-files |grep ambari-server 
7. Reboot server and run #ps -ef|grep ambari-server to verify. 

Note that if ambari-server version is lower than 2.2.2 and RHEL/CentOS 7 version is 7.2 or above then systemd does not work on ambari-server (Hortonworks JIRA BUG-51254). The workaround is to run the following on ambari-server host:

unlink /etc/rc.d/init.d/ambari-server && cp -a /usr/sbin/ambari-server /etc/rc.d/init.d/ambari-server && systemctl daemon-reload 


From https://www.guru99.com/hbase-shell-general-commands.html#2:

Also https://www.guru99.com/handling-tables-hbase.html

After successful installation of HBase on top of Hadoop, we get an interactive shell to execute various commands and perform several operations. Using these commands, we can perform multiple operations on data-tables that can give better data storage efficiencies and flexible interaction by the client.

We can interact with HBase in two ways,

    HBase interactive shell mode and
    Through Java API

In HBase, interactive shell mode is used to interact with HBase for table operations, table management, and data modeling. By using Java API model, we can perform all type of table and data operations in HBase. We can interact with HBase using this both methods.

The only difference between these two is Java API use java code to connect with HBase and shell mode use shell commands to connect with HBase.

Quick overcap of HBase before we proceed-

    HBase uses Hadoop files as storage system to store the large amounts of data. Hbase consists of Master Servers and Regions Servers
    The data that is going to store in HBase will be in the form of regions. Further, these regions will be split up and stored in multiple region servers
    This shell commands allows the programmer to define table schemas and data operations using complete shell mode interaction
    Whichever command we use, it's going to reflect in HBase data model
    We use HBase shell commands in operating system script interpreters like Bash shell
    Bash shell is the default command interpreters for most of Linux and Unix operating distributions
    HBase advanced versions provides shell commands jruby-style object oriented references for tables
    Table reference variables can be used to perform data operations in HBase shell mode

For examples,

    In this tutorial, we have created a table in which 'education' represents table name and corresponds to column name "guru99".
    In some commands "guru99," itself represents a table name.

In this tutorial- you will learn,

    General commands

    Tables Managements commands

    Data manipulation commands

    Cluster Replication Commands

General commands

In Hbase, general commands are categorized into following commands

    Status
    Version
    Table_help ( scan, drop, get, put, disable, etc.)
    Whoami

To get enter into HBase shell command, first of all, we have to execute the code as mentioned below

HBase Shell and General Commands

Once we get to enter into HBase shell, we can execute all shell commands mentioned below. With the help of these commands, we can perform all type of table operations in the HBase shell mode.

Let us look into all of these commands and their usage one by one with an example.

    Status

    Syntax:-hbase(main):001:0>status 

This command will give details about the system status like a number of servers present in the cluster, active server count, and average load value. You can also pass any particular parameters depending on how detailed status you want to know about the system. The parameters can be 'summary', 'simple', or 'detailed', the default parameter provided is "summary".

Below we have shown how you can pass different parameters to the status command.

hbase(main):002:0>status 'simple'

hbase(main):003:0>status 'summary'

hbase(main):004:0> status 'detailed'

If we observe the below screen shot, we will get a better idea.

Syntax: hbase(main):001:0>status

When we execute this command status, it will give information about number of server's present, dead servers and average load of server, here in screenshot it shows the information like- 1 live server, 1 dead servers, and 7.0000 average load.

HBase Shell and General Commands

    Version

    Syntax:hbase(main):005:0> version

    HBase Shell and General Commands
        This command will display the currently used HBase version in command mode
        If you run version command, it will give output as shown above 
    Table help

    Syntax: hbase(main) :007:0>table_help

    HBase Shell and General Commands

    This command guides
        What and how to use table-referenced commands
        It will provide different HBase shell command usages and its syntaxes
        Here in the screen shot above, its shows the syntax to "create" and "get_table" command with its usage. We can manipulate the table via these commands once the table gets created in HBase.
        It will give table manipulations commands like put, get and all other commands information.
    whoami

    Syntax: hbase(main):006:0> Whoami

    HBase Shell and General Commands

    This command "whoami" is used to return the current HBase user information from the HBase cluster.

    It will provide information like
        Groups present in HBase
        The user information, for example in this case "hduser" represent the user name as shown in screen shot

    TTL(Time To Live) - Attribute

    In HBase, Column families can be set to time values in seconds using TTL. HBase will automatically delete rows once the expiration time is reached. This attribute applies to all versions of a row – even the current version too.

    The TTL time encoded in the HBase for the row is specified in UTC. This attribute used with table management commands.

    Important differences between TTL handling and Column family TTLs are below
        Cell TTLs are expressed in units of milliseconds instead of seconds.
        A cell TTLs cannot extend the effective lifetime of a cell beyond a Column Family level TTL setting.

Tables Managements commands

These commands will allow programmers to create tables and table schemas with rows and column families.

The following are Table Management commands

    Create
    List
    Describe
    Disable
    Disable_all
    Enable
    Enable_all
    Drop
    Drop_all
    Show_filters
    Alter
    Alter_status

Let us look into various command usage in HBase with an example.

    Create

    Syntax: hbase> create <tablename>, <columnfamilyname>

    HBase Shell and General Commands

    Example:-

    hbase(main):001:0> create 'education' ,'guru99'

    0 rows(s) in 0.312 seconds

    =>Hbase::Table – education

    The above example explains how to create a table in HBase with the specified name given according to the dictionary or specifications as per column family. In addition to this we can also pass some table-scope attributes as well into it.

    In order to check whether the table 'education' is created or not, we have to use the "list" command as mentioned below.
    List

    Syntax:hbase(main):001:0>list

    HBase Shell and General Commands
        "List" command will display all the tables that are present or created in HBase
        The output showing in above screen shot is currently showing the existing tables in HBase
        Here in this screenshot, it shows that there are total 8 tables present inside HBase
        We can filter output values from tables by passing optional regular expression parameters 
    Describe

    Syntax: hbase>describe <table name>

    HBase Shell and General Commands 

This command describes the named table.

    It will give more information about column families present in the mentioned table
    In our case, it gives the description about table "education."
    It will give information about table name with column families, associated filters, versions and some more details.

    disable

    Syntax: hbase>disable <tablename>

    HBase Shell and General Commands
        This command will start disabling the named table
        If table needs to be deleted or dropped, it has to disable first 

Here, in the above screenshot we are disabling table education

    disable_all

    Syntax:- hbase>disable_all<"matching regex"
        This command will disable all the tables matching the given regex.
        The implementation is same as delete command (Except adding regex for matching)
        Once the table gets disable the user can able to delete the table from HBase
        Before delete or dropping table, it should be disabled first 

    Enable

    Syntax: hbase>enable <tablename>

    HBase Shell and General Commands
        This command will start enabling the named table
        Whichever table is disabled, to retrieve back to its previous state we use this command
        If a table is disabled in the first instance and not deleted or dropped, and if we want to re-use the disabled table then we have to enable it by using this command.
        Here in the above screenshot we are enabling the table "education."
    show_filters

    Syntax:hbase>show_filters

    HBase Shell and General Commands

    This command displays all the filters present in HBase like ColumnPrefix Filter, TimestampsFilter, PageFilter, FamilyFilter, etc.
    drop

    Syntax:hbase>drop <table name>

    HBase Shell and General Commands

    We have to observe below points for drop command
        To delete the table present in HBase, first we have to disable it
        To drop the table present in HBase, first we have to disable it
        So either table to drop or delete first the table should be disable using disable command
        Here in above screenshot we are dropping table "education."
        Before execution of this command, it is necessary that you disable table "education."
    drop_all

    Syntax:Hbase>drop_all<"regex">
        This command will drop all the tables matching the given regex
        Tables have to disable first before executing this command using disable_all
        Tables with regex matching expressions are going to drop from HBase
    is_enabled

    Syntax:hbase>is_enabled 'education'

    This command will verify whether the named table is enabled or not. Usually, there is a little confusion between "enable" and "is_enabled" command action, which we clear here
        Suppose a table is disabled, to use that table we have to enable it by using enable command
        is_enabled command will check either the table is enabled or not
    alter

    Syntax:- hbase> alter <tablename>, NAME=><column familyname>, VERSIONS=>5 

HBase Shell and General Commands

This command alters the column family schema. To understand what exactly it does, we have explained it here with an example.

Examples:

In these examples, we are going to perform alter command operations on tables and on its columns. We will perform operations like

    Altering single, multiple column family names
    Deleting column family names from table
    Several other operations using scope attributes with table 

    To change or add the 'guru99_1' column family in table 'education' from current value to keep a maximum of 5 cell VERSIONS, 

    "education" is table name created with column name "guru99" previously
    Here with the help of an alter command we are trying to change the column family schema to guru99_1 from guru99

hbase> alter 'education', NAME=>'guru99_1', VERSIONS=>5

    You can also operate the alter command on several column families as well. For example, we will define two new column to our existing table "education". 

hbase> alter 'edu', 'guru99_1', {NAME => 'guru99_2', IN_MEMORY => true}, {NAME => 'guru99_3', VERSIONS => 5}

HBase Shell and General Commands

    We can change more than one column schemas at a time using this command
    guru99_2 and guru99_3 as shown in above screenshot are the two new column names that we have defined for the table education
    We can see the way of using this command in the previous screen shot

    In this step, we will see how to delete column family from the table. To delete the 'f1' column family in table 'education'. 

Use one ofthese commands below,

hbase> alter 'education', NAME => 'f1', METHOD => 'delete'

hbase> alter 'education', 'delete' =>' guru99_1'

    In this command, we are trying to delete the column space name guru99_1 that we previously created in the first step 

HBase Shell and General Commands

    As shown in the below screen shots, it shows two steps – how to change table scope attribute and how to remove the table scope attribute. 

Syntax: hbase(main):002:0> alter <'tablename'>, MAX_FILESIZE=>'132545224'

HBase Shell and General Commands

Step 1) You can change table-scope attributes like MAX_FILESIZE, READONLY, MEMSTORE_FLUSHSIZE, DEFERRED_LOG_FLUSH, etc. These can be put at the end;for example, to change the max size of a region to 128MB or any other memory value we use this command.

Usage:

    We can use MAX_FILESIZE with the table as scope attribute as above
    The number represent in MAX_FILESIZE is in term of memory in bytes

NOTE: MAX_FILESIZE Attribute Table scope will be determined by some attributes present in the HBase. MAX_FILESIZE also come under table scope attributes.

Step 2) You can also remove a table-scope attribute using table_att_unset method. If you see the command

Syntax: hbase(main):003:0> alter 'education', METHOD => 'table_att_unset', NAME => 'MAX_FILESIZE'

    The above screen shot shows altered table name with scope attributes
    Method table_att_unset is used to unset attributes present in the table
    The second instance we are unsetting attribute MAX_FILESIZE
    After execution of the command, it will simply unset MAX_FILESIZE attribute from"education" table. 

    alter_status

    Syntax: hbase>alter_status 'education'

    HBase Shell and General Commands
        Through this command, you can get the status of the alter command
        Which indicates the number of regions of the table that have received the updated schema pass table name
        Here in above screen shot it shows 1/1 regions updated. It means that it has updated one region. After that if it successful it will display comment done. 

Data manipulation commands

These commands will work on the table related to data manipulations such as putting data into a table, retrieving data from a table and deleting schema, etc.

The commands come under these are

    Count
    Put
    Get
    Delete
    Delete all
    Truncate
    Scan 

Let look into these commands usage with an example.

    Count

    Syntax: hbase> count <'tablename'>, CACHE =>1000
        The command will retrieve the count of a number of rows in a table. The value returned by this one is the number of rows.
        Current count is shown per every 1000 rows by default.
        Count interval may be optionally specified.
        Default cache size is 10 rows.
        Count command will work fast when it is configured with right Cache.

Example:

HBase Shell and General Commands

    hbase> count 'guru99', CACHE=>1000

This example count fetches 1000 rows at a time from "Guru99" table.

We can make cache to some lower value if the table consists of more rows.

But by default it will fetch one row at a time.

    hbase>count 'guru99', INTERVAL => 100000

    hbase> count 'guru99', INTERVAL =>10, CACHE=> 1000

    If suppose if the table "Guru99" having some table reference like say g.

    We can run the count command on table reference also like below
    hbase>g.count INTERVAL=>100000

    hbase>g.count INTERVAL=>10, CACHE=>1000 

    Put

    Syntax: hbase> put <'tablename'>,<'rowname'>,<'columnvalue'>,<'value'>

    This command is used for following things
        It will put a cell 'value' at defined or specified table or row or column.
        It will optionally coordinate time stamp.

    Example:
        Here we are placing values into table "guru99" under row r1 and column c1

        hbase> put 'guru99', 'r1', 'c1', 'value', 10
        We have placed three values, 10,15 and 30 in table "guru99" as shown in screenshot below 

    HBase Shell and General Commands

        Suppose if the table "Guru99" having some table reference like say g. We can also run the command on table reference also like hbase> g.put 'guru99', 'r1', 'c1', 'value', 10

        The output will be as shown in the above screen shot after placing values into "guru99".

To check whether the input value is correctly inserted into the table, we use "scan" command. In the below screen shot, we can see the values are inserted correctly

HBase Shell and General Commands

Code Snippet: For Practice

create 'guru99', {NAME=>'Edu', VERSIONS=>213423443}

put 'guru99', 'r1', 'Edu:c1', 'value', 10

put 'guru99', 'r1', 'Edu:c1', 'value', 15

put 'guru99', 'r1', 'Edu:c1', 'value', 30

From the code snippet, we are doing these things

    Here we are creating a table named 'guru99' with the column name as "Edu."
    By using "put" command, we are placing values into row name r1 in column "Edu" into table "guru99."

    Get

    Syntax: hbase> get <'tablename'>, <'rowname'>, {< Additional parameters>}

    Here <Additional Parameters> include TIMERANGE, TIMESTAMP, VERSIONS and FILTERS. 

By using this command, you will get a row or cell contents present in the table. In addition to that you can also add additional parameters to it like TIMESTAMP, TIMERANGE,VERSIONS, FILTERS, etc. to get a particular row or cell content.

HBase Shell and General Commands

Examples:-

    hbase> get 'guru99', 'r1', {COLUMN => 'c1'}

    For table "guru99' row r1 and column c1 values will display using this command as shown in the above screen shot
    hbase> get 'guru99', 'r1'

    For table "guru99"row r1 values will be displayed using this command
    hbase> get 'guru99', 'r1', {TIMERANGE => [ts1, ts2]}

    For table "guru99"row 1 values in the time range ts1 and ts2 will be displayed using this command
    hbase> get 'guru99', 'r1', {COLUMN => ['c1', 'c2', 'c3']}

    For table "guru99" row r1 and column families' c1, c2, c3 values will be displayed using this command 

    Delete

    Syntax: hbase> delete <'tablename'>,<'row name'>,<'column name'>
        This command will delete cell value at defined table of row or column.
        Delete must and should match the deleted cells coordinates exactly.
        When scanning, delete cell suppresses older versions of values.

    HBase Shell and General Commands

    Example:
        hbase(main):)020:0> delete 'guru99', 'r1', 'c1''.
        The above execution will delete row r1 from column family c1 in table "guru99."
        Suppose if the table "guru99" having some table reference like say g.
        We can run the command on table reference also like hbase> g.delete 'guru99', 'r1', 'c1'". 
    deleteall

    Syntax: hbase>deleteall <'tablename'>, <'rowname'>

    HBase Shell and General Commands
        This Command will delete all cells in a given row.
        We can define optionally column names and time stamp to the syntax.

    Example:-

    hbase>deleteall 'guru99', 'r1', 'c1'

    hbase>deleteall 'guru99', 'r1', 'c1',

    This will delete all the rows and columns present in the table. Optionally we can mention column names in that.
    Truncate

    Syntax: hbase> truncate <tablename>

    HBase Shell and General Commands

    After truncate of an hbase table, the schema will present but not the records. This command performs 3 functions; those are listed below
        Disables table if it already presents
        Drops table if it already presents
        Recreates the mentioned table 
    Scan

    Syntax: hbase>scan <'tablename'>, {Optional parameters}

    This command scans entire table and displays the table contents.
        We can pass several optional specifications to this scan command to get more information about the tables present in the system.
        Scanner specifications may include one or more of the following attributes.
        These are TIMERANGE, FILTER, TIMESTAMP, LIMIT, MAXLENGTH, COLUMNS, CACHE, STARTROW and STOPROW.

    Examples:-

    The different usages of scan command 

Command
	

Usage

hbase> scan '.META.', {COLUMNS => 'info:regioninfo'}
	

It display all the meta data information related to columns that are present in the tables in HBase

hbase> scan 'guru99', {COLUMNS => ['c1', 'c2'], LIMIT => 10, STARTROW => 'xyz'}
	

It display contents of table guru99 with their column families c1 and c2 limiting the values to 10

hbase> scan 'guru99', {COLUMNS => 'c1', TIMERANGE => [1303668804, 1303668904]}
	

It display contents of guru99 with its column name c1 with the values present in between the mentioned time range attribute value

hbase> scan 'guru99', {RAW => true, VERSIONS =>10}
	

In this command RAW=> true provides advanced feature like to display all the cell values present in the table guru99

hbase(main):016:0> scan 'guru99'

The output as below shown in screen shot

HBase Shell and General Commands

In the above screen shot

    It shows "guru99" table with column name and values
    It consists of three row values r1, r2, r3 for single column value c1
    It displays the values associated with rows 

Code Snippet:-

First create table and place values into table

create 'guru99', {NAME=>'e', VERSIONS=>2147483647}

put 'guru99', 'r1', 'e:c1', 'value', 10

put 'guru99', 'r1', 'e:c1', 'value', 12

put 'guru99', 'r1', 'e:c1', 'value', 14

delete 'guru99', 'r1', 'e:c1', 11

Input Screenshot:

HBase Shell and General Commands

If we run scan command Query:hbase(main):017:0> scan 'guru99', {RAW=>true, VERSIONS=>1000}

It will display output shown in below.

Output screen shot:

HBase Shell and General Commands

The output shown in above screen shot gives the following information

    Scanning guru99 table with attributes RAW=>true, VERSIONS=>1000
    Displaying rows with column families and values
    In the third row, the values displayed shows deleted value present in the column
    The output displayed by it is random; it cannot be same order as the values that we inserted in the table

Cluster Replication Commands

    These commands work on cluster set up mode of HBase.
    For adding and removing peers to cluster and to start and stop replication these commands are used in general.

Command
	

Functionality

add_peer
	

Add peers to cluster to replicate

hbase> add_peer '3', zk1,zk2,zk3:2182:/hbase-prod

remove_peer
	

Stops the defined replication stream.

Deletes all the metadata information about the peer

hbase> remove_peer '1'

start_replication
	

Restarts all the replication features

hbase> start_replication

stop_replication
	

Stops all the replication features

hbase>stop_replication

Summary:

HBase shell and general commands give complete information about different type of data manipulation, table management, and cluster replication commands. We can perform various functions using these commands on tables present in HBase. 
