##  Lab: Apache NiFi

This lab walks you through the process of using the CaptureChangeMySQL, EnforceOrder and PutDatabaseRecord 
processors in Apache NiFi to replicate a database using MySQL event logs.


----

### Binary Log Settings

To enable binary logging on my MySQL instance, I added the following to the [mysqld] section of the my.cnf config file:

    server_id = 1
    log_bin = delta
    binlog_format=row
    binlog_do_db = source

which sets the server ID to 1, the prefix for binlog files to “delta", enables row-level binlog events and logs changes only for the source database in the MySQL instance. In my environment, the my.cnf was located in /usr/local/etc.

Start MySQL and check to see that bin logs are being created (delta.000001, for example). In my installation, the logs were created in /usr/local/var/mysql.
Create Databases and Users Table

Create two databases, "source" and "copy":

    unix> mysql –u root –p
    unix> Enter password:<enter>
    mysql> create database source;
    mysql> create database copy;

Create the table ‘users’ in the source database:

    mysql> use source;
    mysql>CREATE TABLE `users` ( 
    `id` mediumint(9) NOT NULL AUTO_INCREMENT PRIMARY KEY, 
    `title` text, 
    `first` text, 
    `last` text, 
    `street` text, 
    `city` text, 
    `state` text, 
    `zip` text, 
    `gender` text, 
    `email` text, 
    `username` text, 
    `password` text, 
    `phone` text, 
    `cell` text, 
    `ssn` text, 
    `date_of_birth` timestamp NULL DEFAULT NULL, 
    `reg_date` timestamp NULL DEFAULT NULL, 
    `large` text, 
    `medium` text, 
    `thumbnail` text, 
    `version` text, 
    `nationality` text) 
    ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=latin1;

Add data to the ‘users’ table:

    mysql> use source;
    mysql> INSERT INTO `users` (`id`, `title`, `first`, `last`, `street`, `city`, `state`, `zip`, `gender`, `email`, `username`, `password`, `phone`, `cell`, `ssn`, `date_of_birth`, `reg_date`, `large`, `medium`, `thumbnail`, `version`, `nationality`) 
    VALUES (1, 'miss', 'marlene', 'shaw', '3450 w belt line rd', 'abilene', 'florida', '31995', 'F', 'marlene.shaw75@example.com', 'goldenpanda70', 'naughty', '(176)-908-6931', '(711)-565-2194', '800-71-1872', '1991-10-07 00:22:53', '2004-01-29 16:19:10', 'http://api.randomuser.me/portraits/women/67.jpg', 'http://api.randomuser.me/portraits/med/women/67.jpg', 'http://api.randomuser.me/portraits/thumb/women/67.jpg', '0.6', 'US'), 
    (2, 'ms', 'letitia', 'jordan', '2974 mockingbird hill', 'irvine', 'new jersey', '64361', 'F', 'letitia.jordan64@example.com', 'lazytiger614', 'aaaaa1', '(860)-602-3314', '(724)-685-3472', '548-93-7031', '1977-11-14 11:58:01', '2002-02-09 17:04:59', 'http://api.randomuser.me/portraits/women/19.jpg', 'http://api.randomuser.me/portraits/med/women/19.jpg', 'http://api.randomuser.me/portraits/thumb/women/19.jpg', '0.6', 'US'), 
    (3, 'mr', 'todd', 'graham', '5760 spring hill rd', 'garden grove', 'north carolina', '81790', 'M', 'todd.graham39@example.com', 'purplekoala484', 'paintball', '(230)-874-6532', '(186)-529-4912', '362-31-5248', '2006-07-25 05:48:01', '2004-12-05 11:26:34', 'http://api.randomuser.me/portraits/men/39.jpg', 'http://api.randomuser.me/portraits/med/men/39.jpg', 'http://api.randomuser.me/portraits/thumb/men/39.jpg', '0.6', 'US'), 
    (4, 'mr', 'seth', 'martinez', '4377 fincher rd', 'chandler', 'south carolina', '73651', 'M', 'seth.martinez82@example.com', 'bigbutterfly149', 'navy', '(122)-782-5822', '(720)-778-8541', '200-80-9087', '1981-02-28 08:22:49', '2009-08-31 12:42:57', 'http://api.randomuser.me/portraits/men/96.jpg', 'http://api.randomuser.me/portraits/med/men/96.jpg', 'http://api.randomuser.me/portraits/thumb/men/96.jpg', '0.6', 'US'), 
    (5, 'mr', 'guy', 'mckinney', '4524 hogan st', 'iowa park', 'ohio', '24140', 'M', 'guy.mckinney53@example.com', 'blueduck623', 'office', '(309)-556-7859', '(856)-764-9146', '973-37-9077', '1983-11-03 22:02:12', '2003-10-20 07:23:06', 'http://api.randomuser.me/portraits/men/24.jpg', 'http://api.randomuser.me/portraits/med/men/24.jpg', 'http://api.randomuser.me/portraits/thumb/men/24.jpg', '0.6', 'US'), 
    (6, 'ms', 'anna', 'smith', '5047 cackson st', 'rancho cucamonga', 'pennsylvania', '56486', 'F', 'anna.smith74@example.com', 'goldenfish121', 'albion', '(335)-388-7351', '(485)-150-6348', '680-20-6440', '1977-09-05 16:08:05', '2008-07-11 11:09:12', 'http://api.randomuser.me/portraits/women/89.jpg', 'http://api.randomuser.me/portraits/med/women/89.jpg', 'http://api.randomuser.me/portraits/thumb/women/89.jpg', '0.6', 'US'), 
    (7, 'mr', 'johnny', 'johnson', '7250 bruce st', 'gresham', 'new mexico', '83973', 'M', 'johnny.johnson73@example.com', 'crazyduck127', 'toast', '(142)-971-3099', '(991)-131-1582', '683-26-4133', '1988-08-12 14:04:27', '2001-04-30 15:32:34', 'http://api.randomuser.me/portraits/men/78.jpg', 'http://api.randomuser.me/portraits/med/men/78.jpg', 'http://api.randomuser.me/portraits/thumb/men/78.jpg', '0.6', 'US'), 
    (8, 'mrs', 'robin', 'white', '7882 northaven rd', 'orlando', 'connecticut', '40452', 'F', 'robin.white46@example.com', 'whitetiger371', 'elizabeth', '(311)-659-3812', '(689)-468-6420', '960-70-3399', '2003-07-05 13:09:41', '2014-10-01 02:54:46', 'http://api.randomuser.me/portraits/women/82.jpg', 'http://api.randomuser.me/portraits/med/women/82.jpg', 'http://api.randomuser.me/portraits/thumb/women/82.jpg', '0.6', 'US'), 
    (9, 'miss', 'allison', 'williams', '7648 edwards rd', 'edison', 'louisiana', '52040', 'F', 'allison.williams82@example.com', 'beautifulfish354', 'sanfran', '(328)-592-3520', '(550)-172-4018', '164-78-8160', '1983-04-09 08:00:42', '2000-01-01 07:18:54', 'http://api.randomuser.me/portraits/women/16.jpg', 'http://api.randomuser.me/portraits/med/women/16.jpg', 'http://api.randomuser.me/portraits/thumb/women/16.jpg', '0.6', 'US'), 
    (10, 'mrs', 'erika', 'king', '1171 depaul dr', 'addison', 'wisconsin', '50082', 'F', 'erika.king55@example.com', 'goldenbutterfly498', 'chill', '(635)-117-5424', '(662)-110-8448', '122-71-7145', '2003-09-19 07:26:17', '2002-12-31 00:08:43', 'http://api.randomuser.me/portraits/women/52.jpg', 'http://api.randomuser.me/portraits/med/women/52.jpg', 'http://api.randomuser.me/portraits/thumb/women/52.jpg', '0.6', 'US');

Note: My data came from the test data site, [RandomUser](https://randomuser.me). This useful site provides a free API to pull down data: https://randomuser.me/api/0.6/?results=10&format=SQL.

At this point, we have bin log data that captures the creation of the ‘users’ table in the "source" database and the insert of rows into the table.

### NIFI Configuration

Start NiFi and upload the following template:

cdc-mysql-replication.xml

You should see the following flow on your NiFi canvas:

### Flow Configuration

There are properties and components that need to be added/adjusted to complete the flow.

Select the first processor in the flow, CaptureChangeMySQL. Right-click and select "Configure" from the context menu. Modify the database related properties to point to the "source" database of your MySQL instance, your local JDBC driver, and add the root password.

Click on the canvas to select the root process group ("NiFi Flow"). Click the "Configuration" button (gear icon) from the Operate palette. This opens the NiFi Flow Configuration window. Select the "Controller Services" tab:

Click the "+" button and add the DistributedMapCacheServer controller service.

No changes need to be made to the default properties.

Select the "MySQL CDC Backup" controller service and select the Edit button (pencil icon). Modify the database properties to point to the "copy" database of your MySQL instance. Add the location of your JDBC driver and the root user password.

Enable the five existing controller services by selecting the Enable button (lightning icon) for each. (Note: JsonPathReader cannot be enabled unless the controller service it references, AvroSchemaRegistry, is enabled first.)

With these updates, only the two LogAttribute processors in the flow should have warnings. We can ignore them for now as they exist for debugging purposes.

We are now ready to run the CDC flow.

### Review

This tutorial walked you through enabling binlog in your MySQL instance, adding two databases and a table to generate bin log data, and importing/configuring a sample NiFi CDC flow. You have a "source" database that contains a ‘users’ table and an empty "copy" database that will be used to replicate the ‘users’ table and any other changes to the source. Continue to the second article for a detailed walk through of the flow.
