# INSTALL - VERIFY MYSQL ENTERPRISE EDITION  

## Introduction

Goal:
    Verify the new MySQL Installation on Linux and import test databases

Objectives:

- understand better how MySQL connection works
- have a look on useful statements

Estimated Time: -- 10 minutes

### Objectives

In this lab, you will:

- Connect to Ports
- Learn Useful SQL Statements

### Prerequisites

This lab assumes you have:

- All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>  
  The command must be executed in the Operating System shell
* ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>  
  The command must be executed in a client like MySQL, MySQL Shell or similar tool
* ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>  
  The command must be executed in MySQL shell

## Task 1: MySQL Connection

Please note that password can be saved in an obfuscated file, to be reused, using the utility mysql_config_editor.  
This approach let you configure more secure and flexible scripts.

1. Set the login path **local_admin** to be used for easier connections 

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql_config_editor set --login-path=local_admin --user=admin --host=127.0.0.1 -p</copy>
    ```

2. Test the connection with mysql and mysqlsh clients

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql --login-path=local_admin</copy>
    ```
 **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>** 
    ```
    <copy>exit</copy>
    ```

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysqlsh --login-path=local_admin</copy>
    ```
 **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>\quit</copy>
    ```

2. List existing settings, please note that password are obfuscated

 **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql_config_editor print --all</copy>
    ```

## Task 2: Learn Useful SQL Statements

1. Connect to your instance

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

2. You can check the values of a specific variable, like the version of your server to know if it's updated to last release or not  
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>SHOW VARIABLES LIKE "%version%";</copy>
    ```

3. InnoDB provides the best Storage Engine in the general use case. it's also the one required for ReplicaSet and InnoDB Cluster. You can check if all there are tables in a different format (of course, excluding system schemas) using this query

    * First let's create a MyISAM table
    ```
    <copy>CREATE TABLE employees.pets (id INT, pet varchar(50)) ENGINE=myisam;</copy>
    ```
    ```
    <copy>INSERT INTO employees.pets values(1,'dog'),(2,'cat'),(2,'bunny');</copy>
    ```
    ```
    <copy>SELECT * FROM employees.pets;</copy>
    ```

    * Then search the non-InnoDB tables
    ```
    <copy>SELECT table_schema table_name, engine
    FROM INFORMATION_SCHEMA.TABLES 
    WHERE engine <> 'InnoDB' AND table_schema NOT IN ('mysql','information_schema', 'sys', 'performance_schema', 'mysql_innodb_cluster_metadata');</copy>
    ```

    * Now we can convert the table to innoDB format
    ```
    <copy>ALTER TABLE employees.pets ENGINE=innodb;</copy>
    ```

4. It's a best practice and a requirement to have a Primary Key in each table. You can search tables without PK with this query
    ```
    <copy>SELECT i.TABLE_ID, t.NAME
    FROM INFORMATION_SCHEMA.INNODB_INDEXES i JOIN INFORMATION_SCHEMA.INNODB_TABLES t ON (i.TABLE_ID = t.TABLE_ID)
    WHERE i.NAME='GEN_CLUST_INDEX';</copy>
    ```

5. You can now exit to continue our labs
    ```
    <copy>\q</copy>
    ```

## Learn More

* [MySQL Tutorial](https://dev.mysql.com/doc/en/tutorial.html)

## Acknowledgements

- **Author** - Dale Dasker, MySQL Solution Engineering
- **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
