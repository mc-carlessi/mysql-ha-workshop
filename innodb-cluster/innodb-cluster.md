# MySQL InnoDB Cluster

## Introduction
An InnoDB Cluster consists of at least three MySQL Server instances, and it provides high-availability and scaling features.

In this lab you will create 3 instances MySQL InnoDB Cluster as Single Primary and have a trial on the MySQL Shell to configure and operate. And you will be using MySQL Router to test for Server routing and test for Failover.


Estimated Lab Time: 45 minutes

### Objectives
In this lab, you will:
* Check and, if required, fix data model for InnoDB Cluster compatibility
* Configure InnoDB Cluster in single primary (mysql1 as primary)
* Configure mysql router
* Test client connection with MySQL Router
* Simulate a crash of primary instance

> **Note:**
 * MySQL InnoDB cluster require at least 3 instances (refer to schema in the first part of the lab guide) 
    * mysql1 will be the primary
    * mysql2 and mysql3 will be first secondaries

## Task 1: Check data model
1. If not already connected, from **app-srv**, connect the client to mysql1 and verify data model compatibility with Group replication requirements. If needed, fix the errors
    
    <span style="color:green">shell></span>
    ```bash
    <span style="color:green">shell-app-srv$</span> <copy>mysqlsh admin@mysql1</copy>
    ```

2. Search non InnoDB tables and if there are you must change them. We don't have them in our sample database, so it's an empty result.
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>SELECT table_schema, table_name, engine, table_rows, (index_length+data_length)/1024/1024 AS sizeMB
    FROM information_schema.tables
    WHERE engine != 'innodb'
    AND table_schema NOT IN ('information_schema', 'mysql', 'performance_schema');</copy>
    ```

3. Search InnoDB tables without primary or unique key. In production you must fix.
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>SELECT i.TABLE_ID, t.NAME
    FROM INFORMATION_SCHEMA.INNODB_INDEXES i JOIN INFORMATION_SCHEMA.INNODB_TABLES t ON (i.TABLE_ID = t.TABLE_ID)
    WHERE i.NAME='GEN_CLUST_INDEX';</copy>
    ```

    > ** OUTPUT SAMPLE. Please note that table-id may be different in your instance 
    
    ```
    +----------+----------------+
    | TABLE_ID | NAME           |
    +----------+----------------+
    |     1070 | employees/pets |
    +----------+----------------+
    ```

4. We created a table without primary key (employees.pets). Now we fix fix it. Becasue we have a column id, but it's not unique, we add a PK inside an invisible column

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>desc employees.pets;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>alter table employees.pets add column added_pk bigint unsigned NOT NULL primary key auto_increment invisible;</copy>
    ```

5. Now let's see the new table structure

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>desc employees.pets;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>show create table employees.pets\G</copy>
    ```

5. Now let's play with the table

    * The new column is not visible with a generic query

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>Select * from employees.pets;</copy>
        ```

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>insert into employees.pets values(5,'canary');</copy>
        ```

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>Select * from employees.pets;</copy>
        ```

    * But you can read it with an explicit reference

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>Select *,added_pk from employees.pets;</copy>
        ```

5. To automatically fix the creation of tables without primary key, we can also enable GTID mode (Generated Primary Invisible Keys).
    We set the variable on mysql1 only, but in the real life remember to ***set in all the instances***.

    * We enable the GIPK mode
        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>SELECT @@sql_generate_invisible_primary_key;</copy>
        ```

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>SET PERSIST sql_generate_invisible_primary_key=1;</copy>
        ```

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>\q</copy>
        ```

    * Now we create a table without primary key

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>CREATE TABLE employees.test_table (c1 VARCHAR(50), c2 INT);</copy>
        ```

        <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
        ```sql
        <copy>SHOW CREATE TABLE employees.test_table\G</copy>
        ```

## Task 2: Prepare instances for the cluster

1. First we dissolve the existing ReplicaSet. Remember to switch to \js mode and retrieve ReplicaSet connection  

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\js</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>rs=dba.getReplicaSet()</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>rs.dissolve()</copy>
    ```

   > **OUTPUT: FINAL PART**

    ```
    ...
    The ReplicaSet was successfully dissolved.
    Replication was disabled but user data was left intact.
    ```

2. Now we install mysql3 like mysql1 and mysql2.
   If you are not yet connected, open a second SSH client to app-srv
    
    <span style="color:green">shell></span>
    ```bash
    <copy>ssh -i <private_key_file> opc@<your_compute_instance_ip></copy>
    ```

3. Connect to <span style="color:red">mysql3</span> server through app-srv
    
    <span style="color:green">shell></span>
    ```bash
    <copy>ssh -i $HOME/sshkeys/id_rsa_mysql3 opc@mysql3</copy>
    ```

4. Install MySQL Server and MySQL Shell using rpms
    
    <span style="color:green">shell></span>
    ```bash
    <copy>sudo yum -y install /workshop/linux/MySQL_server_rpms/*.rpm</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo yum -y install /workshop/linux/mysql-shell*.rpm</copy>
    ```

5.	Start your new mysql instance

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo systemctl start mysqld</copy>
    ```

6.	Retrieve root password for first login:

    <span style="color:green">shell></span> 
    ```bash
    <copy>sudo grep -i 'temporary password' /var/log/mysqld.log</copy>
    ```

7. Login to the the mysql-enterprise and change temporary password

    <span style="color:green">shell></span> 
     ```bash
    <copy>mysqlsh root@localhost</copy>
    ```
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

8.	Create a new administrative user called 'admin' with remote access and full privileges

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>CREATE USER 'admin'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;</copy>
    ```

9. Close the SSH session to mysql3 and return to app-srv. 
   **Keep** this connection **open on app-srv** to use it later on

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\q</copy>
    ```

    <span style="color:green">shell></span> 
     ```bash
    <copy>exit</copy>
    ```

10. Return to administrative connection and check mysql1 configuration (we may have or not errors)

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>dba.checkInstanceConfiguration('admin@mysql1')</copy>
    ```

    > **OUTPUT EXAMPLE with CONFIGURATION ERRORS**

    ```
    ...
    NOTE: Some configuration options need to be fixed:
    +----------------------------------------+---------------+----------------+----------------------------+
    | Variable                               | Current Value | Required Value | Note                       |
    +----------------------------------------+---------------+----------------+----------------------------+
    | binlog_transaction_dependency_tracking | COMMIT_ORDER  | WRITESET       | Update the server variable |
    +----------------------------------------+---------------+----------------+----------------------------+

    NOTE: Please use the dba.configureInstance() command to repair these issues.

    {
        "config_errors": [
            {
                "action": "server_update",
                "current": "COMMIT_ORDER",
                "option": "binlog_transaction_dependency_tracking",
                "required": "WRITESET"
            }
        ],
        "status": "error"
    }
    ```

11. In case of errors, we use MySQL Shell to fix them.
   We now **force the reconfiguration** to create the a dedicated account just for cluster administration (please accept all changes and the reboot) 

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>dba.configureInstance('admin@mysql1', {clusterAdmin: "'csadmin'@'%'"});</copy>
    ```

    > **OUTPUT EXAMPLE: FIX ERRORS**

    ```
    Configuring local MySQL instance listening at port 3307 for use in an InnoDB cluster...

    This instance reports its own address as mysql1
    Clients and other cluster members will communicate with it through this address by default. If this is not correct, the report_host MySQL system variable should be changed.

    applierWorkerThreads will be set to the default value of 4.

    NOTE: Some configuration options need to be fixed:
    +----------------------------------------+---------------+----------------+----------------------------+
    | Variable                               | Current Value | Required Value | Note                       |
    +----------------------------------------+---------------+----------------+----------------------------+
    | binlog_transaction_dependency_tracking | COMMIT_ORDER  | WRITESET       | Update the server variable |
    +----------------------------------------+---------------+----------------+----------------------------+

    Do you want to perform the required configuration changes? [y/n]: y
    Configuring instance...

    WARNING: '@@binlog_transaction_dependency_tracking' is deprecated and will be removed in a future release. (Code 1287).
    The instance 'mysql1' was configured to be used in an InnoDB cluster.
    ```

   **OUTPUT EXAMPLE: NO CONFIGURATION ERRORS**
    ```text
    Configuring MySQL instance at mysql1:3306 for use in an InnoDB ReplicaSet...

    This instance reports its own address as mysql1:3306
    Clients and other cluster members will communicate with it through this address by default. If this is not correct, the report_host MySQL system variable should be changed.
    Password for new account: *********
    Confirm password: *********

    applierWorkerThreads will be set to the default value of 4.

    Creating user csadmin@%.
    Account csadmin@% was successfully created.


    The instance 'mysql1:3306' is valid for InnoDB ReplicaSet usage.

    Successfully enabled parallel appliers.
    ```

12. Repeat the command on mysql2 and mysql3

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>dba.configureInstance('admin@mysql2', {clusterAdmin: "'csadmin'@'%'"});</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>dba.configureInstance('admin@mysql3', {clusterAdmin: "'csadmin'@'%'"});</copy>
    ```

## Task 3: Create Cluster 

1. It's now time to create the cluster. Connect to mysql1 with the new account
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>\connect csadmin@mysql1</copy>
    ```

2. We now create the cluster using the non default option **EVENTUAL** for primary failover.
   The default is **BEFORE\_ON\_PRIMARY\_FAILOVER**, choose the option that better fits your SLA requirements

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>var cluster = dba.createCluster("myCluster", {consistency:'EVENTUAL', paxosSingleLeader:true})</copy>
     ```

   **OUTPUT EXAMPLE: CLUSTER CREATED**

    ```
    A new InnoDB Cluster will be created on instance 'mysql1'.

    Validating instance configuration at mysql1...

    This instance reports its own address as mysql1

    Instance configuration is suitable.
    NOTE: Group Replication will communicate with other members using 'mysql1'. Use the localAddress option to override.

    * Checking connectivity and SSL configuration...

    Creating InnoDB Cluster 'testCluster' on 'mysql1'...

    Adding Seed Instance...
    Cluster successfully created. Use Cluster.addInstance() to add MySQL instances.
    At least 3 instances are needed for the cluster to be able to withstand up to
    one server failure.
    ```


3. Verify cluster status (why "Cluster is NOT tolerant to any failures" ?)
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.status()</copy>
    ```

   **OUTPUT EXAMPLE: CLUSTER STATUS WITH ONE INSTANCE**

    ```
    {
        "clusterName": "testCluster",
        "defaultReplicaSet": {
            "name": "default",
            "primary": "mysql1:3306",
            "ssl": "REQUIRED",
            "status": "OK_NO_TOLERANCE",
            "statusText": "Cluster is NOT tolerant to any failures.",
            "topology": {
                "mysql1": {
                    "address": "mysql1:3306",
                    "memberRole": "PRIMARY",
                    "mode": "R/W",
                    "readReplicas": {},
                    "replicationLag": "applier_queue_applied",
                    "role": "HA",
                    "status": "ONLINE",
                    "version": "8.4.3"
                }
            },
            "topologyMode": "Single-Primary"
        },
        "groupInformationSourceMember": "mysql1:3306"
    }
    ```

4.  Add mysql2 instance to the cluster.
    Please note that this instance has a compatible data set, so the join possibly uses binary logs to align it
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.addInstance('mysql2')</copy>
    ```

   **OUTPUT EXAMPLE: ADD SECOND INSTANCE (EXTRACT)**

    ```
    The safest and most convenient way to provision a new instance is through automatic clone provisioning, which will completely overwrite the state of 'mysql2' with a physical snapshot from an existing cluster member. To use this method by default, set the 'recoveryMethod' option to 'clone'.

    The incremental state recovery may be safely used if you are sure all updates ever executed in the cluster were done with GTIDs enabled, there are no purged transactions and the new instance contains the same GTID set as the cluster or a subset of it. To use this method by default, set the 'recoveryMethod' option to 'incremental'.

    Incremental state recovery was selected because it seems to be safely usable.

    ...

    The instance 'mysql2' was successfully added to the cluster.
    ```
 

5. Verify cluster status

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.status()</copy>
    ```

   **OUTPUT EXAMPLE: CLUSTER STATUS WITH 2 INSTANCES**
    ```
    {
        "clusterName": "testCluster",
        "defaultReplicaSet": {
            "name": "default",
            "primary": "mysql1:3306",
            "ssl": "REQUIRED",
            "status": "OK_NO_TOLERANCE",
            "statusText": "Cluster is NOT tolerant to any failures.",
            "topology": {
                "mysql1": {
                    "address": "mysql1:3306",
                    "memberRole": "PRIMARY",
                    "mode": "R/W",
                    "readReplicas": {},
                    "replicationLag": "applier_queue_applied",
                    "role": "HA",
                    "status": "ONLINE",
                    "version": "8.4.3"
                },
                "mysql2": {
                    "address": "mysql2:3306",
                    "memberRole": "SECONDARY",
                    "mode": "R/O",
                    "readReplicas": {},
                    "replicationLag": "applier_queue_applied",
                    "role": "HA",
                    "status": "ONLINE",
                    "version": "8.4.3"
                }
            },
            "topologyMode": "Single-Primary"
        },
        "groupInformationSourceMember": "mysql1:3306"
    }
    ```

6. Add the third instance to cluster. We see here how to assign a different weight, here lower than the default 50, to make this node the last choice in case of automatic election

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.addInstance('mysql3', {memberWeight:30})</copy>
    ```

    **OUTPUT EXAMPLE: ADD 3 INSTANCE GTID WARNING (EXTRACT)**

    ```
    WARNING: A GTID set check of the MySQL instance at 'mysql3' determined that it contains transactions that do not originate from the cluster, which must be discarded before it can join the cluster.

    mysql3 has the following errant GTIDs that do not exist in the cluster:
    ...
    ```

    * To complete this step choose **[C]lone** as recovery mode to use the MySQL clone plugin

   **OUTPUT EXAMPLE: ADD 3 INSTANCE WITH CLONE (EXTRACT)**

    ```
    Please select a recovery method [C]lone/[A]bort (default Abort): C
    Validating instance configuration at mysql3...

    ...

    The instance 'mysql3' was successfully added to the cluster.
    ```

7. Verify cluster status, now with all three servers. Please check that **"status": "OK"** and **"primary": "mysql1"**
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.status()</copy>
    ```

   **OUTPUT EXAMPLE: CLUSTER STATUS WITH 3 INSTANCES**
    ```
    {
        "clusterName": "testCluster",
        "defaultReplicaSet": {
            "name": "default",
            "primary": "mysql1:3306",
            "ssl": "REQUIRED",
            "status": "OK",
            "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
            "topology": {
                "mysql1": {
                    "address": "mysql1:3306",
                    "memberRole": "PRIMARY",
                    "mode": "R/W",
                    "readReplicas": {},
                    "replicationLag": "applier_queue_applied",
                    "role": "HA",
                    "status": "ONLINE",
                    "version": "8.4.3"
                },
                "mysql2": {
                    "address": "mysql2:3306",
                    "memberRole": "SECONDARY",
                    "mode": "R/O",
                    "readReplicas": {},
                    "replicationLag": "applier_queue_applied",
                    "role": "HA",
                    "status": "ONLINE",
                    "version": "8.4.3"
                },
                "mysql3": {
                    "address": "mysql3:3306",
                    "memberRole": "SECONDARY",
                    "mode": "R/O",
                    "readReplicas": {},
                    "replicationLag": "applier_queue_applied",
                    "role": "HA",
                    "status": "ONLINE",
                    "version": "8.4.3"
                }
            },
            "topologyMode": "Single-Primary"
        },
        "groupInformationSourceMember": "mysql1:3306"
    }
    ```

8. It's also possible to increase details from "1" to "3" using {extended: ... } option
    
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.status({extended:3})</copy>
    ```

   **OUTPUT EXAMPLE: PARTIAL**
    ```
{
    "clusterName": "myCluster",
    "defaultReplicaSet": {
        "GRProtocolVersion": "8.0.27",
        "communicationStack": "MYSQL",
        "groupName": "3bcfacdb-8bc5-11ef-9e85-0200170cbf80",
        "groupViewChangeUuid": "AUTOMATIC",
        "groupViewId": "17290864703141694:33",
        "name": "default",
        "paxosSingleLeader": "OFF",
        "primary": "mysql3:3306",
        "ssl": "REQUIRED",
        "status": "OK",
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
        "topology": {
            "mysql1:3306": {
                "address": "mysql1:3306",
                "applierWorkerThreads": 4,
                "fenceSysVars": [
                    "read_only",
                    "super_read_only"
                ],
                "memberId": "246491d8-87d0-11ef-8159-0200170cbf80",
                "memberRole": "SECONDARY",
                "memberState": "ONLINE",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": "applier_queue_applied",
                "role": "HA",
                "status": "ONLINE",
                "transactions": {
                    "appliedCount": 6956,
                    "checkedCount": 6956,
                    "committedAllMembers": "246491d8-87d0-11ef-8159-0200170cbf80:1-540,3bcfacdb-8bc5-11ef-9e85-0200170cbf80:1-12402:1000077-1000123,f9985ea2-87dc-11ef-9bde-020017006b87:1-9942",
                    "conflictsDetectedCount": 0,
                    "connection": {
                        "lastHeartbeatTimestamp": "",
                        "lastQueued": {
                            "endTimestamp": "2024-10-17 10:07:05.697356",
                            "immediateCommitTimestamp": "2024-10-17 10:07:05.696312",
                            "immediateCommitToEndTime": 0.001044,
                            "originalCommitTimestamp": "2024-10-17 10:07:05.696312",
                            "originalCommitToEndTime": 0.001044,
                            "queueTime": 0.000057,
                            "startTimestamp": "2024-10-17 10:07:05.697299",
                            "transaction": "3bcfacdb-8bc5-11ef-9e85-0200170cbf80:12417"
    ...
    ```

9. Now cluster is up and running, and we test it


## Task 4: update MySQL Router to be used with cluster, and test it

1.  Return to the second shell and reconfigure MySQL Router top be used with the new cluster. Because there is already a configuration, we override it with --force option.

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo mysqlrouter --bootstrap csadmin@mysql1 --user=mysqlrouter --force</copy>
    ```

2. Restart MySQL Router

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo systemctl restart mysqlrouter</copy>
    ```

3. Test the connection on port 6446 port (read/write).

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh appuser@127.0.0.1:6446</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>SELECT @@hostname;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>select * from employees.pets;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>insert into employees.pets values(6,'turtle');</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>select * from employees.pets;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\q</copy>
    ```

4. Like for ReplicaSet, we can use the RO port (6447) for reads. RO ports is used in round robin, so let's test it executing the same read query multiple times (plese notes the server where we are connected)

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh appuser@127.0.0.1:6447 --mysql --table -e 'select @@hostname'</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh appuser@127.0.0.1:6447 --mysql --table -e 'select @@hostname'</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh appuser@127.0.0.1:6447 --mysql --table -e 'select @@hostname'</copy>
    ```

5. We can now test the failover. Connect to RW port.
   We use here the standard mysql client that provides reconnecting options

    <span style="color:green">shell></span>
    ```bash
    <copy>mysql -u appuser -p -h127.0.0.1 -P6446</copy>
    ```

    <span style="color:blue">mysql></span>
    ```
    <copy>SELECT @@hostname;</copy>
    ```

6. Return to administrative connection, and use it to kill the instance on mysql1.
   To execute the kill command, use the mysqld process ID in stored in /var/run/mysqld/mysqld.pid file (here we can do it in a single command)

    <span style="color:green">shell></span>
    ```bash
    <copy>\q</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo kill -9 $( sudo cat /var/run/mysqld/mysqld.pid )</copy>
    ```

7. Now <span style="color:red">return to mysql connection previously opened</span> and verify that reconnection happens. It may requires 15/20 seconds to be work, so repeat the command multiple times.

    <span style="color:blue">mysql></span>
    ```sql
    <copy>SELECT @@hostname;</copy>
    ```

    <span style="color:blue">mysql></span>
    ```sql
    <copy>SELECT @@hostname;</copy>
    ```

8. Return From the shell where you killed the instance use MySQL Shell to verify cluster status. Because mysql1 was killed, you need to reconnect

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh csadmin@mysql1</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\js</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>var cluster = dba.getCluster()</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.status()</copy>
    ```

9. We can also test the read/write split port (**6450**).
    To check the full rules see [here](https://dev.mysql.com/doc/mysql-router/9.0/en/router-read-write-splitting-statements.html).
    Return to appuser connection and close it

    <span style="color:blue">mysql></span>
    ```sql
    <copy>\q</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh appuser@127.0.0.1:6450</copy>
    ```

10. Check where read only query are executed

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>select @@hostname;</copy>
    ```

11. Check where "write" transaction are executed

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>START TRANSACTION;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>select @@hostname;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>COMMIT;</copy>
    ```

## Task 5: Connect PHP application to router

1. From your local  machine connect to dbtest.php. Note the instance where you are now connected

    Example: http://129.213.167..../dbtest.php

2. Return to admin mysqlsh and make mysql1 the new primary

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.setPrimaryInstance('mysql1')</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>cluster.status()</copy>
    ```

3. From your local  machine connect to dbtest.php. Note that you are **immediately** connected to mysql1

    Example: http://129.213.167..../dbtest.php

4. Repeat failover test.
   Return to shell, verify that you are on mysql1 and kill the MySQL Server instance.

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:yellow">JS</span>>
    ```js
    <copy>\q</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>hostname</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo kill -9 $( sudo cat /var/run/mysqld/mysqld.pid )</copy>
    ```

5. As soon as possible, from your local machine, refresh dbtest.php page. ***Please wait*** that page reconnect to the new primary

    Example: http://129.213.167..../dbtest.php

6. This is the end of the workshop


## Acknowledgements
* **Author** - Marco Carlessi, Principal Sales Consultant
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
