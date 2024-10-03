# PERFORMANCE - MYSQL ENTERPRISE SCALABILITY (THREAD POOLING)

## Introduction
Performance enhancements
MySQL Enterprise Scalability provides an easy to use, built-in database solution to increase performance during heavy periods of application connections.
Objective: Install and use Thread Pooling functionalities

Estimated Lab Time: -- 40 minutes

### Objectives

In this lab, you will:
* Install and test out sysBench
* Install BMK Toolkit
* Run benchmarks with increasing number of connections
* Install Thread Pool
* Run benchmarks with increasing number of connections

### Prerequisites

This lab assumes you have:
* An Oracle account
* A basic understanding of Cloud Computing.

* Lab standard  
    - ![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell> the command must be executed in the Operating System shell
    - ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) mysql> the command must be executed in a client like MySQL, MySQL Workbench
    - ![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh> the command must be executed in MySQL shell
    

**Notes:**
- Thread Pooling has more functions than what we test in the lab. The full list of functions is here
- https://dev.mysql.com/doc/en/thread-pool.html

**Presentation**
- [Presentation](files/MySQL_ThreadPool.pdf)


## Task 1: Install and setup environment.  These steps can be handled on your own time if you are not able to keep up.    

1.	Setup hardware and networking as described in Lab 1 & Lab 2

    a. Follow steps listed below.  For Lab 2 where you create the Compute, use a **32 OCPU Core** with **1TB of storage** with VPU at 20.  It is typically recommended to use a secondary server for running the benchmarks client and the primary server for running MySQL.  For this workshop I am running them all on one for convenience.  If you use these steps to test your environment, please setup a client server to run the benchmarks on.

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
*    [Lab 1](https://ddasker.github.io/mysql-enterprise/workshops/freetier/index.html?lab=create-vcn)
*    [Lab 2](https://ddasker.github.io/mysql-enterprise/workshops/freetier/index.html?lab=create-compute)

2. SSH to Server 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>ssh -i id_rsa opc@public_ip </copy>
    ```

3.  Make /workshop Directory

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mkdir workshop </copy>
    ```

4.  Download workshop files 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>cd workshop </copy>
    ```

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo wget https://objectstorage.us-ashburn-1.oraclecloud.com/p/UnCA6ATQ3Oea4GgEeGZyR2TU4sghesB7BR0cUf8xz_oBIbO34jP3qkry8Ky1WI_n/n/idazzjlcjqzj/b/bucket-20230816-0811j-ThreadPool/o/threadpool.tar</copy>
    ```
    
5.  Expand filesystem to use all available space

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo /usr/libexec/oci-growfs</copy>
    ```

6.  Extract workshop files 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>tar xvf threadpool.tar</copy>
    ```

7.	Install EPEL Repository and sysbench

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm</copy>
    ```

8.	Install all RPMs and Setup MySQL:

    a. Run Yum 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo yum -y install *.rpm</copy>
    ```

    b. Start MySQL daemon

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo systemctl start mysqld</copy>
    ```

    c. Retrieve root password for first login: 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
    ```
    <copy>sudo grep -i 'temporary password' /var/log/mysqld.log</copy>
    ```

    d. Login to the the mysql-enterprise installation and check the status (you will be asked to change password)

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>** 
     ```
    <copy>mysqlsh --uri root@localhost:3306 --sql -p </copy>
    ```

    e. Create New Password for MySQL Root

    **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Welcome1!';</copy>
    ```

     **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\status</copy>
    ```

     **![#ff9933](https://via.placeholder.com/15/ff9933/000000?text=+) mysqlsh>**
    ```
    <copy>\quit</copy>
    ```

9.	Setup OS for handling large amount of connections:

    a. Edit the /etc/security/limits.conf

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo nano /etc/security/limits.conf</copy>
    ```

    b. Add the following lines

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy> *                soft    nofile          131072
    *                hard    nofile          131072
    *                soft    nproc           65536
    *                hard    nproc           65536</copy>    
    ```

    c. Setup OS for more open files 

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo sysctl net.ipv4.tcp_max_syn_backlog=30000
    sudo sysctl net.core.netdev_max_backlog=30000
    sudo sysctl net.core.somaxconn=30000</copy>
    ```

    d. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```

10.	Setup the benchmarking 

    a. Create the sample database

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    
    ```
    <copy>mysql -uroot -pWelcome1! -e"CREATE DATABASE sysbench"</copy>
    ```

    b. Create the benchmark user

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql -uroot -pWelcome1!</copy>
    ```
    ```
    <copy>CREATE USER 'dim'@'%' IDENTIFIED WITH mysql_native_password BY 'Welcome1!';</copy>
    <copy>GRANT ALL ON *.* TO 'dim'@'%';</copy>
    <copy>quit;</copy>
    ```

    c. Setup MySQL to handle many connections

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo cp /home/opc/workshop/my.conf-TPX512-tp32-w2 /etc/my.cnf</copy>
    <copy>sudo vi /etc/my.cnf</copy>
    ```
    ```
    <copy>skip-log-bin</copy>
    ```
    ```
    <copy>max_connections=25000</copy>
    ```

    d. Determine what file to edit if using systemd for MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld status</copy>
    ```

    e. Increase number of threads possible for the mysqld daemon

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo vi /usr/lib/systemd/system/mysqld.service</copy>
    ```

    e. Modify LimitNOFILE under [Service] section

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>LimitNOFILE=50000</copy>
    ```

    f. Reload Daemons

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo systemctl daemon-reload</copy>
    ```

    g. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```



## Task 2: Install and setup Benchmarks  
1.	Install BMK toolkit

    a. Untar files

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>cd ~
    tar xvf workshop/BMK-kit.tgz</copy>
    ```

    b. Setup environment variables

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>vi ~/.bashrc</copy>
    ```

    c. Add environment variables

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>export BMK_HOME="/home/opc/BMK"</copy>
    ```

    d. Change login details for Toolkit

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>vi ~/BMK/.bench</copy>
    ```

    e. Make it look like this:

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    user=dim
    pass=Welcome1!
    host=127.0.0.1
    port=3306
    ```

2.	Initialize and start BMK toolkit

    a. Prepare sysbench table

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>time bash /home/opc/BMK/sb_exec/sb11-Prepare_TPCC_100W-InnoDB-NoFK.sh 32</copy>
    ```

3.	Create bash script for iterating through number of connections

    a. Entitle the file bench.sh

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>vi bench.sh</copy>
    ```

    b. Contents should be

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>#!/bin/bash
    for users in 1 2 4 8 16 32 64 128 256 512 1024 2048 4000 5000 6000 8000 9000 10000 11000 12000 13000 14000 15000 16000 17000 18000 19000 20000
    do
        bash /home/opc/BMK/sb_exec/sb11-TPCC_100W.sh $users 300 --luajit-cmd=off
    sleep 30
    done</copy>
    ```

    c. Change the mod of the file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>chmod +x bench.sh</copy>
    ```

4.	Run benchmarks

    a. Run bench.sh and copy the output to a file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>./bench.sh | tee output_No_TP.txt</copy>
    ```

    b. Sit back and wait 2.5 hours

## Task 3: Install, setup, and run Benchmarks with Thread Pool enabled 

1.	Install the Thread Pool plugin

    a. **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    
    ```
    <copy>sudo nano /etc/my.cnf</copy>
    ```

    b. Uncomment the following lines to load the plugin

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    # TP
    plugin_load=thread_pool=thread_pool.so
    thread_pool_max_transactions_limit=512
    thread_pool_size=32
    thread_pool_query_threads_per_group=2
    thread_pool_algorithm=1 
    ```

    c. Restart MySQL

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>sudo service mysqld restart</copy>
    ```

    d. Confirm that the plugin has been loaded
    
    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>mysql -uroot -pWelcome1! -e"SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'thread%';"</copy>
    ```

2.	Run benchmarks

    a. Run bench.sh and copy the output to a file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```
    <copy>./bench.sh | tee output_With_TP.txt</copy>
    ```
    b. Sit back and wait 2.5 hours

## Task 4: My results


| # of Connections 	|QPS    |TPS 	|QPS(TP) 	|TPS(TP)|
|-------------------|-------|-------|-----------|-------|
| 1	                |3948	|136	|**4277**	    |**146**   |
| 2	                |12498	|430	|**12433**	    |**429**  |
| 4	                |25748	|886	|**25313**	    |**872**  |
| 8	                |47091	|1622	|**46149**	    |**1590**  |
| 16	            |77443	|2670	|**75161**	    |**2586** |
| 32	            |128157	|4417	|**119583**	    |**4118**   |
| 64	            |212434	|7319	|**196392**	    |**6762** |
| 128	            |277523	|9557	|**265636**	    |**9148** |
| 256	            |257896	|8883	|**262621**	    |**9043** |
| 512	            |250025	|8613	|**251548**	    |**8661** |
| 1024	            |213924	|7372	|**223456**	    |**7696**   |
| 2048	            |158844	|5472	|**211762**	    |**7295**   |
| 4096	            |98837	|3400	|**230035**	    |**7924**   |
| 5000	            |59822	|2063	|**236715**	    |**8150**   |
| 6000	            |59870	|2064	|**236008**	    |**8133**   |
| 7000	            |60709	|2089	|**237729**	    |**8185**   |
| 8000	            |60177	|2072	|**239878**	    |**8258**   |
| 9000	            |56948	|1957	|**246243**	    |**8482**   |
| 9500	            |54150	|1862	|**245960**	    |**8469**   |
| 10000	            |51865	|1785	|**242213**	    |**8340**   |
| 11000	            |48113	|1660	|**243037**	    |**8359**   |
| 12000	            |45636	|1576	|**241805**	    |**8322**   |
| 13000	            |42762	|1472	|**246141**	    |**8480**   |
| 14000	            |40150	|1382	|**245191**	    |**8444**   |
| 15000	            |37326	|1288	|**245158**	    |**8444**   |
| 16000	            |35588	|1226	|**243007**	    |**8367**   |
| 18000	            |32173	|1108	|**240250**	    |**8279**   |
| 19000	            |30877	|1065	|**235391**	    |**8111**   |
| 20000	            |29645	|1027	|**232043**	    |**7991**   |

## Raw Results
[Raw Results](files/Summation_of_Results.pdf)

## Learn More

* [Thread Pool Plugins](https://dev.mysql.com/doc/en/thread-pool.html)
* [sysBench code](https://github.com/akopytov/sysbench/tree/master)
* [sysBench useage](https://www.mortensi.com/2021/06/how-to-install-and-use-sysbench/)
* [BMK Toolkit useage](http://dimitrik.free.fr/blog/posts/mysql-perf-bmk-kit.html)

## Acknowledgements
* **Author** - Dale Dasker, MySQL Solution Engineering
* **Last Updated By/Date** - Dale Dasker, August 2023
