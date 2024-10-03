# SECURITY - DATA MASKING

## Introduction
Data Masking and de-identification
MySQL Enterprise Masking and De-identification provides an easy to use, built-in database solution to help organizations protect sensitive data from unauthorized uses by hiding and replacing real values with substitutes.
Objective: Install and use data masking functionalities

Estimated Lab Time: -- 12 minutes

### Objectives

In this lab, you will:
* Create sample data with random generation utilites which are part of Enterprise Masking
* Test Masking of Sensitive Data
* Create a View and user which only sees masked data

### Prerequisites

This lab assumes you have:
* An Oracle account
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
- Data masking has more functions than what we test in the lab. The full list of functions is here
- https://dev.mysql.com/doc/en/data-masking-usage.html 

## Task 1: Install masking plugin

1. To install the data masking plugin, log with administrative account and using mysql standard protocol  
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysqlsh admin@127.0.0.1 --mysql</copy>
    ```

2. Create the masking_dictionaries table  
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>CREATE TABLE IF NOT EXISTS
            mysql.masking_dictionaries(
            Dictionary VARCHAR(256) NOT NULL,
            Term VARCHAR(256) NOT NULL,
            UNIQUE INDEX dictionary_term_idx (Dictionary, Term),
            INDEX dictionary_idx (Dictionary)
            ) ENGINE = InnoDB DEFAULT CHARSET=utf8mb4;</copy>
    ```

3. Load and install the masking components  
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>INSTALL COMPONENT 'file://component_masking';</copy>
    ```

    ```
    <copy>INSTALL COMPONENT 'file://component_masking_functions';</copy>
    ```

4. Check if the components are loaded  
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>SELECT * FROM mysql.component;</copy>
    ```

## Task 2: Use masking functions (examples)

1. Use data masking functions

    a. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>SELECT mask_inner(last_name, 2,1) FROM employees.employees limit 10;</copy>
    ```

    b. **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>SELECT mask_outer(last_name, 2,1) FROM employees.employees limit 10;</copy>
    ```

2. Generate numbers between 1 and 200  
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>SELECT gen_range(1, 200);</copy>
    ```

3. Generate email where name has 4 characters, surname has 5 characters and domain is mynet.com  
    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>SELECT gen_rnd_email(4, 5, 'mynet.com');</copy>
    ```


## Task 3: Use Masking functions to create and hide data

1. Create Table to generate and add masking data

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>** 
    ```
    <copy>USE employees; CREATE TABLE employees_mask LIKE employees;</copy>
    ```

2. Add data to newly created table

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>INSERT INTO employees_mask SELECT * FROM employees;</copy>
    ```

3. Create new column for SSN's

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>ALTER TABLE employees_mask ADD COLUMN ssn varchar(11);</copy>
    ```

4. Create new column for emails's

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>ALTER TABLE employees_mask ADD COLUMN email varchar(40);</copy>
    ```

5. Use Functions to generate sample SSN data

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>UPDATE employees_mask SET ssn = gen_rnd_ssn() WHERE 1;</copy>
    ```

6. Use Functions to generate sample Email data

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>UPDATE employees_mask SET email = gen_rnd_email() WHERE 1;</copy>
    ```

7. Let's look at the data that we just created

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>SELECT * FROM employees_mask LIMIT 5;</copy>
    ```

8. Let's mask the SSN

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>SELECT emp_no,first_name,last_name,mask_ssn(CONVERT(ssn USING latin1)) AS ssn FROM employees_mask LIMIT 5;</copy>
    ```

9. Let's create a view which only shows the masked data

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>CREATE VIEW masked_customer AS SELECT emp_no,first_name,last_name,mask_ssn(CONVERT(ssn USING latin1)) AS ssn FROM employees_mask;</copy>
    ```

10. Let's create a user who only has access to the view with the masked data

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**  
    ```
    <copy>CREATE USER 'accounting'@'%' IDENTIFIED BY 'Pa33word!';</copy>
    ```

    ```
    <copy>GRANT SELECT ON employees.masked_customer TO 'accounting'@'%';</copy>
    ```

    ```
    <copy>\quit</copy>
    ```

11. Log in with new user account and run queries


    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>mysql -uaccounting -pPa33word! -h 127.0.0.1</copy>
    ```

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>SELECT * FROM employees.masked_customer LIMIT 5;</copy>
    ```

12. Try accessing table that is not masked

    **![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql>**  
    ```
    <copy>SELECT * FROM employees.employees_mask LIMIT 5;</copy>
    ```

13. Run the application to see the SSN and Email data:

    http://computeIP/emp_apps/employees_mask.php

14. Run the application to see the masked SSN data:

    http://computeIP/emp_apps/employees_mask_after.php


You may now **proceed to the next lab**

## Learn More

* [Enterprise Data Masking Documentation](https://dev.mysql.com/doc/en/data-masking.html)
* [Whitepaper: A MySQL Guide to PCI Compliance](https://www.mysql.com/why-mysql/white-papers/mysql-pci-data-security-compliance/)
* [Whitepaper: MySQL Enterprise and the GDPR](https://www.mysql.com/why-mysql/white-papers/mysql-enterprise-edition-gdpr/)
* [Whitepaper: MySQL Secure Deployment Guide](https://dev.mysql.com/doc/mysql-secure-deployment-guide/en/)

## Acknowledgements

* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
