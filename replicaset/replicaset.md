# MySQL Replication

## Introduction

Replication enables data from one MySQL database server (known as a source) to be copied to one or more MySQL database servers (known as replicas). 

In this lab you will prepare and create a replica.

Estimated Lab Time: 25 minutes

### Objectives
In this lab, you will:
* Install binaries on a new server
* Have that server act as a Replica


## Task 1: Prepare replica server (mysql2)

> **Note:** Please be sure that you work on teh right server

1. If you are not yet connected, open an SSH client to app-srv
    ```
    <span style="color:green">shell></span> <copy>ssh -i <private_key_file> opc@<your_compute_instance_ip></copy>
    ```

2. Connect to <span style="color:red">mysql2</span> server through app-srv
    ```
    <span style="color:green">shell-app-srv$</span> <copy>ssh -i $HOME/sshkeys/id_rsa_mysql2 opc@mysql2</copy>
    ```

3. Install MySQL Server and MySQL Shell using rpms
    ```
    <span style="color:green">shell-mysql2></span> <copy>sudo yum -y install /workshop/linux/MySQL_server_rpms/*.rpm</copy>
    ```
    ```
    <span style="color:green">shell-mysql2></span> <copy>sudo yum -y install /workshop/linux/mysql-shell*.rpm</copy>
    ```

4.	Start your new mysql instance

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

5.	Retrieve root password for first login:

  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo grep -i 'temporary password' /var/log/mysqld.log</copy>
    ```

6. Login to the the mysql-enterprise and change temporary password

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>mysqlsh root@localhost</copy>
    ```
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

7.	Create a new administrative user called 'admin' with remote access and full privileges

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>CREATE USER 'admin'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;</copy>
    ```

8. Close the SSH session to mysql2 and return to app-srv. 
   **Keep** this connection **open on app-srv** to use it later on


## Task 2: Install MySQL client on app-srv
1. Using the existing SSH connection, install theMySQL Client on app-srv
    ```
    <span style="color:green">shell></span> <copy>ssh -i <private_key_file> opc@<your_compute_instance_ip></copy>
    ```

3. Install MySQL client and MySQL Shell using rpms
    ```
    <span style="color:green">shell-mysql2></span> <copy>ls -l /workshop/linux/client/</copy>
    ```
    ```
    <span style="color:green">shell-mysql2></span> <copy>sudo yum install -y /workshop/linux/client/*.rpm</copy>
    ```

4. Keep this SSH session open

## Task 3: Create ReplicaSet
> **Note:**
 * Servers: 
    * mysql1 (source)
    * mysql2 (replica)
 * Some commands must run inside the source, other on replica: please read carefully the instructions

1. Open MySQL Shell, enable history.save and switch to javascript command mode
    ```
    <span style="color:green">shell-app-srv$</span> <copy>mysqlsh</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>\option --persist history.autoSave=true</copy>
    ```
    ```
    <span style="color:blue">mysql></span> <copy>\js</copy>
    ```

2. Configure primary (source) and secondary (replica) instances to be used for replication, with a dedicated account (suggested password 'Welcome1!').  

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>dba.configureReplicaSetInstance('admin@mysql1', {clusterAdmin: "'rsadmin'@'%'"});</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>dba.configureReplicaSetInstance('admin@mysql2', {clusterAdmin: "'rsadmin'@'%'"});</copy>
    ```

> Output

    ```
    Password for new account: *********
    Confirm password: *********

    applierWorkerThreads will be set to the default value of 4.

    Creating user rsadmin@%.
    Account rsadmin@% was successfully created.


    The instance 'mysql3:3306' is valid for InnoDB ReplicaSet usage.

    Successfully enabled parallel appliers.
    ```

4. Connect to primary instance as rsadmin

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\c rsadmin@mysql1</copy>
    ```

5. Create the ReplicaSet

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>var rs = dba.createReplicaSet("myreplica")</copy>
    ```

> Output
    ```
    A new replicaset with instance 'mysql1:3306' will be created.

    * Checking MySQL instance at mysql1:3306

    This instance reports its own address as mysql1:3306
    mysql1:3306: Instance configuration is suitable.

    * Checking connectivity and SSL configuration...
    * Updating metadata...

    ReplicaSet object successfully created for mysql1:3306.
    Use rs.addInstance() to add more asynchronously replicated instances to this replicaset and rs.status() to check its status.
    ```

6. Check replica status
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>rs.status()</copy>
    ```

> Output
    ```
    {
        "replicaSet": {
            "name": "myreplica",
            "primary": "mysql1:3306",
            "status": "AVAILABLE",
            "statusText": "All instances available.",
            "topology": {
                "mysql1:3306": {
                    "address": "mysql1:3306",
                    "instanceRole": "PRIMARY",
                    "mode": "R/W",
                    "status": "ONLINE"
                }
            },
            "type": "ASYNC"
        }
    }
    ```

7. Add mysql2 as replica instance for the ReplicaSet. Please note that clone is used to copy data from mysql1 to mysql2, and that the user to connect to mysql2 is the one of the actual connection

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>rs.addInstance('mysql2')</copy>
    ```

> Partial Output
    ```
    Adding instance to the replicaset...

    * Performing validation checks

    This instance reports its own address as mysql2:3306
    mysql2:3306: Instance configuration is suitable.

    ...

    The instance 'mysql2:3306' was added to the replicaset and is replicating from mysql1:3306.

    * Waiting for instance 'mysql2:3306' to synchronize the Metadata updates with the PRIMARY...
    ** Transactions replicated  ############################################################  100%
    ```

8. Check the new replica status
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>rs.status()</copy>
    ```

> Output  

    ```
    {
        "replicaSet": {
            "name": "myreplica",
            "primary": "mysql1:3306",
            "status": "AVAILABLE",
            "statusText": "All instances available.",
            "topology": {
                "mysql1:3306": {
                    "address": "mysql1:3306",
                    "instanceRole": "PRIMARY",
                    "mode": "R/W",
                    "status": "ONLINE"
                },
                "mysql2:3306": {
                    "address": "mysql2:3306",
                    "instanceRole": "SECONDARY",
                    "mode": "R/O",
                    "replication": {
                        "applierStatus": "APPLIED_ALL",
                        "applierThreadState": "Waiting for an event from Coordinator",
                        "applierWorkerThreads": 4,
                        "receiverStatus": "ON",
                        "receiverThreadState": "Waiting for source to send event",
                        "replicationLag": null,
                        "replicationSsl": "TLS_AES_128_GCM_SHA256 TLSv1.3",
                        "replicationSslMode": "REQUIRED"
                    },
                    "status": "ONLINE"
                }
            },
            "type": "ASYNC"
        }
    }
    ```

9. Keep MySQL Shell session open

## Task 4: Install MySQL Router and test ReplicaSet

1. Please use the second SSH session to app-srv, or open a second one if you closed it

2. Install MySQL Router on app-srv
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>sudo yum install -y /workshop/linux/mysql-router*.rpm</copy>
    ```

3. Bootstrap MySQL Router. Insert rsadmin password when requested.
   We are now using the default ports.

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>sudo mysqlrouter --bootstrap rsadmin@mysql1 --user=mysqlrouter</copy>
    ```

4. Start router

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>sudo systemctl start mysqlrouter</copy>
    ```

5. Verify MySQL Router listening ports

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>netstat -an | grep 644</copy>
    ```

6. Install on app-srv the MySQL client and MySQL Shell to test MySQL Router

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>sudo yum install -y /workshop/linux/client/*.rpm</copy>
    ```

7. Connect to read/write port and verify how it works

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>mysqlsh appuser@127.0.0.1:6446</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select @@hostname;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select * from employees.pets;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>insert into employees.pets values(3,'hamster');</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select * from employees.pets;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\q</copy>
    ```

8. Connect to read only port and verify how it works (please note that writes return an error)

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>mysqlsh appuser@127.0.0.1:6447</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select @@hostname;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select * from employees.pets;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>insert into employees.pets values(4,'t-rex');</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select * from employees.pets;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\q</copy>
    ```

9. Now we test primary/secondary switchover. First Reconnect to read/write port

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>mysqlsh appuser@127.0.0.1:6446</copy>
    ```

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>select @@hostname;</copy>
    ```

9. Return to admin mysqlsh and switch primary and secondary

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>rs.setPrimaryInstance('mysql2')</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>rs.status()</copy>
    ```

10. Switch to appuser connection and retry to use it (first connection fails)

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select @@hostname;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select @@hostname;</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>insert into employees.pets values(4,'horse');</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>select * from employees.pets;</copy>
    ```

## Task 5: Connect PHP application to router

1. Edit dbtest.php application and configure it to use localhost and port 6446

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo nano /var/www/html/dbtest.php</copy>
    ```

2. From your local  machine connect to dbtest.php. Note that you are connected to mysql2

    Example: http://129.213.167..../dbtest.php

3. Return to admin mysqlsh and switch primary and secondary

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>rs.setPrimaryInstance('mysql1')</copy>
    ```

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>rs.status()</copy>
    ```
4. From your local  machine connect to dbtest.php. Note that you are connected to mysql1

    Example: http://129.213.167..../dbtest.php



## Acknowledgements
* **Author** - Marco Carlessi, MySQL Solution Engineering
* **Contributors** -  Perside Foster, MySQL Solution Engineering, Selena Sánchez, MySQL Solutions Engineer
* **Last Updated By/Date** - Selena Sánchez, MySQL Solution Engineering, May 2023
