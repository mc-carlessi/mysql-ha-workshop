# INSTALL - MYSQL ENTERPRISE EDITION

## Introduction

Detailed Installation of MySQL Enterprise Edition 8 and MySQL Shell on Linux

Objective: RPM Installation of MySQL 8 Enterprise on Linux


RPM Installation of MySQL Enterprise 8 on Linux

Estimated Time: 15 minutes

### Objectives

In this lab, you will:

* Install MySQL Enterprise Edition
* Install MySQL Shell and Connect to MySQL Enterprise 
* Start and test MySQL Enterprise Edition Install


### Prerequisites

This lab assumes you have:
* A working Oracle Linux machine
* The MySQL Enterprise rpms for Oracle Linux inside /workshop directory


### Lab standard

Pay attention to the prompt, to know where execute the commands 


## Task 1: Install MySQL Enterprise Edition using Linux RPM's

1. Connect to **app-srv** instance

    <span style="color:green">shell></span>
    ```bash
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_compute_instance_ip></copy>
    ```

    ![CONNECT](./images/ssh-login-2.png " ")

2. We install MySQL Server in a dedicated machine. 

  * Let's connect to mysql1 server

    <span style="color:green">shell></span>
    ```bash
    <copy>ssh -i $HOME/sshkeys/id_rsa_mysql1 opc@mysql1</copy>
    ```

  * verify that you are connected to mysql1

    <span style="color:green">shell></span>
    ```bash
    <copy>hostname</copy>
    ```

  We have the required software available in **/workshop** directory.

3. Now install the RPM's from /workshop/linux/MySQL_server_rpms.

    <span style="color:green">shell></span>
    ```bash
    <copy>cd /workshop/linux/MySQL_server_rpms</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>ls -l</copy>
    ```

    <span style="color:green">shell></span>
    ```
    <copy>sudo yum -y install *.rpm</copy>
    ```

4. Now we install the new MySQL Shell client rpm.

    <span style="color:green">shell></span>
    ```bash
    <copy>cd /workshop/linux/</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>ls -l mysql-shell*.rpm</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo yum -y install mysql-shell*.rpm</copy>
    ```


## Task 2: Start and test MySQL Enterprise Edition Install


1.	Start your new mysql instance

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo systemctl start mysqld</copy>
    ```

2.	Verify that process is running and listening on the default ports (3306 for MySQL standard protocol and 33060 for MySQL XDev protocol)

    <span style="color:green">shell>
    ```bash
    </span> <copy>ps -ef | grep mysqld</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>netstat -an | grep 3306</copy>
    ```


3.	Another way is searching the message “ready for connections” in error log as one of the last 

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo grep -i ready /var/log/mysqld.log</copy>
    ```

4.	Retrieve root password for first login:

    <span style="color:green">shell></span>
    ```bash
    <copy>sudo grep -i 'temporary password' /var/log/mysqld.log</copy>
    ```

5. Login to the the mysql-enterprise, change temporary password and check instance the status (***DON'T SAVE THE PASSWORD***)

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh root@localhost</copy>
    ```

6. Create New Password for MySQL Root

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\status</copy>
    ```

7.	Create a new administrative user called 'admin' with remote access and full privileges

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>CREATE USER 'admin'@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'%' WITH GRANT OPTION;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\quit</copy>
    ```

8.	Login as the new user, save the password. MySQL Shell save the password in a secure file (mysql_config_editor is the default) and set history autosave

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\option -l</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\option --persist history.autoSave true</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\option history.autoSave</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\quit</copy>
    ```

## Task 3: Import Sample Databases

1. Import the employees demo database that is in /workshop/databases folder.

    <span style="color:green">shell></span>
    ```bash
    <copy>cd /workshop/databases/employees/</copy>
    ```

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh admin@127.0.0.1 < ./employees.sql</copy>
    ```

2. Create now a generic user with access to our sample database

    <span style="color:green">shell></span>
    ```bash
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>CREATE USER appuser@'%' IDENTIFIED BY 'Welcome1!';</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>GRANT ALL ON employees.* TO appuser@'%';</copy>
    ```

## Task 4: Learn Useful SQL Statements

1. You can check the values of a specific variable, like the version of your server to know if it's updated to last release or not  

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>SHOW VARIABLES LIKE "%version%";</copy>
    ```

2. InnoDB provides the best Storage Engine in the general use case. it's also the one required for ReplicaSet and InnoDB Cluster. You can check if all there are tables in a different format (of course, excluding system schemas) using this query

  * First let's create a MyISAM table

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>CREATE TABLE employees.pets (id INT, pet varchar(50)) ENGINE=myisam;</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>INSERT INTO employees.pets values(1,'dog'),(2,'cat'),(2,'bunny');</copy>
    ```

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>SELECT * FROM employees.pets;</copy>
    ```

  * Then search the non-InnoDB tables

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>SELECT table_schema table_name, engine
    FROM INFORMATION_SCHEMA.TABLES 
    WHERE engine <> 'InnoDB' AND table_schema NOT IN ('mysql','information_schema', 'sys', 'performance_schema', 'mysql_innodb_cluster_metadata');</copy>
    ```

  * Now we can convert the table to innoDB format
  
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>ALTER TABLE employees.pets ENGINE=innodb;</copy>
    ```

3. It's a best practice and a requirement to have a Primary Key in each table. You can search tables without PK with this query
  
    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>SELECT i.TABLE_ID, t.NAME
    FROM INFORMATION_SCHEMA.INNODB_INDEXES i JOIN INFORMATION_SCHEMA.INNODB_TABLES t ON (i.TABLE_ID = t.TABLE_ID)
    WHERE i.NAME='GEN_CLUST_INDEX';</copy>
    ```

4. You can now exit from MySQL Shell

    <span style="color:blue">My</span><span style="color: orange">SQL </span><span style="background-color:orange">SQL</span>>
    ```sql
    <copy>\q</copy>
    ```

5. Close now your connection to mysql1, to return on app-srv

    <span style="color:green">shell></span>
    ```bash
    <copy>exit</copy>
    ```

You may now **proceed to the next lab**

## Learn More

* [MySQL Linux Installation](https://dev.mysql.com/doc/en/binary-installation.html)
* [MySQL Shell Installation](https://dev.mysql.com/doc/mysql-shell/en/mysql-shell-install.html)

## Acknowledgements

* **Author** - Marco Carlessi, MySQL Solution Engineering
* **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
