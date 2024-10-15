# Setup LAMP Application

## Introduction

MySQL Enterprise Edition  can easily be used for development tasks. Applications can be created with the LAMP or other software stacks.

**Note:** This application code is intended for educational purposes only. It is designed to help developers learn and practice application development skills with MySQL Enterprise Edition. The code is not designed to be used in a production environment

_Estimated Lab Time:_ 15 minutes

### Objectives

In this lab, you will be guided through the following tasks:

- Install Apache and PHP
- Create PHP / MYSQL Connect Application
- Migrate LAMP WEB Application

### Prerequisites

- All previous labs successfully completed

## Task 1: Install App Server (APACHE) on app-srv

> NOTE: in previous lab we worked on mysql1 server, now we install the application on app-srv

1. If not already connected with SSH, on Command Line, connect to the Compute instance using SSH ... be sure replace the  "private key file"  and the "new compute instance ip"

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
     ```bash
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_compute_instance_ip></copy>
     ```

2. Verofy that you are now working on app-srv

    ```bash
    <copy>hostname</copy>
    ```

3. Install app server

    a. Install Apache

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo yum install httpd -y </copy>
    ```

    b. Enable Apache

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo systemctl enable httpd</copy>
    ```

    c. Start Apache

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo systemctl restart httpd</copy>
    ```

4. From a browser test apache from your local machine using the Public IP Address of your Compute Instance

    **Example: http://129.213....**

## Task 2: Install PHP

1. Install php:

    a. Install php:7.4

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy> sudo dnf module install php:7.4 -y</copy>
    ```

    b. Install php libraries required by our application

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo yum install php-cli php-mysqlnd php-zip php-gd php-mbstring php-xml php-json -y</copy>
    ```

    c. View installed php libraries for mysql

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>php -m |grep mysql</copy>
    ```

    d. View php version

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>php -v</copy>
    ```

    e. Restart Apache

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo systemctl restart httpd</copy>
    ```

2. Create test php file (info.php)

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo nano /var/www/html/info.php</copy>
    ```

3. Add the following code to the editor and save the file (ctr + o) (ctl + x)

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy><?php
    phpinfo();
    ?></copy>
    ```

4. From your local machine, browse the page info.php

   Example: http://129.213.167.../info.php

## Task 3: Create MySQL PHP connect app

1. Create dbtest.php file

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>sudo nano dbtest.php</copy>
    ```

2. Add the following code to the editor and save the file (ctr + o) (ctl + x)

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>
    <!DOCTYPE html>
    <html>
    <body>

    <?php
    $DB_SERVER='mysql1:3306';
    $DB_USERNAME='appuser';
    $DB_PASSWORD='Welcome1!';
    $DB_NAME='employees';

    $retries = 20;

    $i=0;
    while ($i <= $retries) {
    $link = mysqli_connect($DB_SERVER, $DB_USERNAME, $DB_PASSWORD, $DB_NAME);
    if($link === false){
        if($i <= $retries) {
        sleep(1);
        $i++;
        } else {
        break;
        }
    } else {
        echo 'Connection info: <b>' . mysqli_get_host_info($link) .'</b><br>';
        echo "Retries: " . $i . "<br>";
        $query = "SELECT @@hostname";
        if ($stmt = $link->prepare($query)) {
            $stmt->execute();
            $stmt->bind_result($hostname);
            $stmt->fetch();
            echo "Hostname: <b>" . $hostname ."</b><br>";
        }
        $stmt->close();
        break;
    }

    if($link === false){
    die("ERROR: Could not connect. " . mysqli_connect_error());
    }

    }
    ?>

    </body>
    </html>
    </copy>
    ```

3. From your local  machine connect to dbtest.php. Application just return connection info

    Example: http://129.213.167..../dbtest.php  



You may now **proceed to the next lab**

## Acknowledgements

- **Author** - Perside Foster, MySQL Solution Engineering
- **Last Updated By/Date** - Perside Foster, MySQL Solution Engineering, August 2024