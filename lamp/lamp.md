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

2. Verify that you are now working on app-srv

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

    ```bash
    <span style="color:green">shell></span> <copy>sudo nano dbtest.php</copy>
    ```

2. Add the following code to the editor and save the file (ctr + o) (ctl + x)

    **![#00cc00](https://via.placeholder.com/15/00cc00/000000?text=+) shell>**
    ```bash
    <copy>
    <!DOCTYPE html>
    <html>
    <body>

    <?php
    // Connection info
    $servername = "127.0.0.1:6446";
    $username = "appuser";
    $password = "Welcome1!";
    $dbname = "employees";

    // Connection to database function
    function connectToDatabase() {
        global $servername, $username, $password, $dbname;
        $conn = null;
        $max_retries = 15;
        $retry_delay = 2; // secondi

        for ($i = 0; $i < $max_retries; $i++) {
            try {
                $conn = new mysqli($servername, $username, $password, $dbname);
                if ($conn->connect_error) {
                    throw new Exception("Connection failed: " . $conn->connect_error);
                }
                break; // If connection is fine, exit from the loop
            } catch (Exception $e) {
                echo "Connection failed (temptative " . ($i+1) . "): " . $e->getMessage() . "<br>";
                sleep($retry_delay);
            }
        }

        return $conn;
    }

    // Call connection function
    $conn = connectToDatabase();

    if ($conn) {
        echo "<br>";
        echo 'Connection info: <b>' . mysqli_get_host_info($conn) .'</b><br>';
        $query = "SELECT @@hostname";
        if ($stmt = $conn->prepare($query)) {
            $stmt->execute();
            $stmt->bind_result($hostname);
            $stmt->fetch();
            echo "Hostname: <b>" . $hostname ."</b><br>";
        }

        $conn->close();
    } else {
        echo "Impossible to connect to database";
    }

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