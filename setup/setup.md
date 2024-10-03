# SETUP Environment

## Introduction

Objective: Connect Personal Computer to the Oracle Network and the Oracle Cloud Infrastructure (OCI)

In this lab you will connect your Personal Computer to the  VM server

Estimated Lab Time: -- 5 minutes

### Objectives

In this lab, you will:

* Connect to your server and Setup workshop directory on Server

### Prerequisites

* *In compliance with Oracle security policies, I acknowledge I will not load actual confidential customer data or Personally Identifiable Information (PII) into my demo server*

* All previous labs successfully completed

  
## Task 1: Setup workshop directory on Server

1. SSH to Server

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_compute_instance_ip></copy>
    ```

    ![MDS](./images/ssh-login-2.png "ssh-login")



2. Make /workshop Directory

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>sudo mkdir /workshop </copy>
    ```

3. Assign ownership to opc user

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>sudo chown opc. /workshop </copy>
    ```


3. Download workshop files

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>cd /workshop </copy>
    ```

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/gXmOYNBcgAWQeWnYuftAXwB7Jd1Iqgr2oTuPZLl2Ekn53iwgap8r60qsK2NGq0Qz/n/idazzjlcjqzj/b/bucket-20240214-SecurityWorkshop03142024/o/Workshop84.tar</copy>
    ```

4. Extract workshop files

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**

    ```
    <copy>tar xvf Workshop84.tar </copy>
    ```

You may now **proceed to the next lab**

## Learn More

* [Creating SSH Keys](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/managingkeypairs.htm)
* [Compute SSH Connections](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/accessinginstance.htm)

## Acknowledgements

* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024
