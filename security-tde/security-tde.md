# SECURITY - MYSQL ENTERPRISE TRANSPARENT DATA ENCRYPTION

## Introduction

3c) MySQL Enterprise Transparent Data Encryption
Objective: Data Encryption in action…

This lab will walk you through encrypting InnoDB Tablespace files at rest

Estimated Lab Time: 20 minutes

### Objectives

In this lab, you will:

* Setup TDE using file encryption component
* Practice using TDE

### Prerequisites (Optional)

This lab assumes you have:

* All previous labs successfully completed

### Lab standard

Pay attention to the prompt, to know where execute the commands 
* ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>  
  The command must be executed in the Operating System shell
* ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>  
  The command must be executed in a client like MySQL, MySQL Shell or similar tool
* ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>  
  The command must be executed in MySQL shell

**Notes:**
    - [InnoDB Data At Rest](https://dev.mysql.com/doc/en/innodb-data-encryption.html)

## Task 1: Setup required files for TDE

1. SSH into server intance and Create the global manifest file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>cd /usr/sbin </copy>
    ```

    ```
    <copy>sudo nano mysqld.my</copy>
    ```

2. copy the following  content to mysqld.my save and exit

    ```
    <copy>
    {
        "components": "file://component_keyring_encrypted_file"
    }
    </copy>
    ```

3. Create the global configuration file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>cd /usr/lib64/mysql/plugin/</copy>
    ```

    ```
    <copy>sudo nano component_keyring_encrypted_file.cnf</copy>
    ```

4. copy the following  content to component\_keyring\_encrypted\_file.cnf save and exit

    ```  
    <copy> 
    {
        "path": "/var/lib/mysql-keyring/keyring-encrypted",
        "password": "Welcome1!",
        "read_only": false
    } </copy>
    ```

5. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>sudo service mysqld restart</copy>
    ```

## Task 2: Use TDE

1. "Spy" on employees.employees table

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```

2. Open another terminal window.

    Now with <span style="color:red">Administrative Account</span> we enable Encryption on the employees.employees table:

    a.  **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>mysqlsh admin@127.0.0.1</copy>
    ```

    b. Verify the component is loaded and active: **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SELECT * FROM performance_schema.keyring_component_status;</copy>
    ```

    c. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>USE employees;</copy>
    ```

    d. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>ALTER TABLE employees ENCRYPTION = 'Y';</copy>
    ```

3. From a second session, "spy" on employees.employees table again:

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>sudo strings "/var/lib/mysql/employees/employees.ibd" | head -n50</copy>
    ```

4. Administrative commands

    a. Get details on encrypted key file:

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SHOW VARIABLES LIKE 'keyring_encrypted_file_data'\G</copy>
    ```

    b. Set default for all tables to be encrypted when creating them:
    
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SET GLOBAL default_table_encryption=ON;</copy>
    ```

    c. Peek on the mysql System Tables:

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

    d. Encrypt the mysql System Tables:

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>ALTER TABLESPACE mysql ENCRYPTION = 'Y';</copy>
    ```

    e. Show all the encrypted tables:

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**

    ```
    <copy>SELECT SPACE, NAME, SPACE_TYPE, ENCRYPTION FROM INFORMATION_SCHEMA.INNODB_TABLESPACES WHERE ENCRYPTION='Y'\G</copy>
    ```

5. Validate encryption of the mysql System Tables:

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>sudo strings "/var/lib/mysql/mysql.ibd" | head -n70</copy>
    ```

6. Run the application to see if it was affected by TDE:

    <http://computeIP/emp_apps/list_employees.php>

You may now **proceed to the next lab**

## Learn More

* [Keyring Plugins](https://dev.mysql.com/doc/en/keyring.html)
* [InnoDB Data At Rest](https://dev.mysql.com/doc/en/innodb-data-encryption.html)

## Acknowledgements

* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
