# Aker-Systems-Project-2-LEMP-Stack-Implementation


## Intro

In this project, I will employ a LAMP  (Linux, Nginx, MySQL, PHP) stack project in an amazon EC2 instance. Here are the steps I took to implement this.


### Initiate EC2 & Install nginx

- An aws ec2 was started and following command was run to install nginx, `sudo apt update && sudo apt install nginx && sudo systemctl status nginx`. 
- Nginx is active and running, html can also seen below when accessed locally via `curl http://127.0.0.1:80`.
- This was also tested on the browser and yeiled successfull results, the inbound security groups were changed to alllow http get this working.

![alt text](images/nginx_install_check.png)

![alt text](images/nginx_active.png)

![alt text](images/nginx%20_web.png)


### MySQL Install

- Next we move on to our DMB install, and we do that via `sudo apt install mysql-server`, we then access this through `sudo mysql` to ensure this is successfull.

![alt text](images/mysql_install_access.png)


### Installing PHP

- PHP was installed and checked via `sudo apt install php libapache2-mod-php php-mysql`, and `php -v` respectively.

![alt text](images/php_install.png)


### Configure nginx to use php processor

- Firstly we are going to create a root web directory for our domain called projectLEMP. This was done through `sudo mkdir /var/www/projectLEMP`, along with assigning the ownership to the system user via `sudo chown -R $USER:$USER /var/www/projectLEMP`.
- Next we created our configuration file within sites-directory via vim using the command `sudo vim /etc/nginx/sites-available/projectLEMP`.

- In the configuration file we placed the following configuration seen below.

```
#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}
```
- We then activate our configuration, by linking it from the sites-available directory `sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`.


![alt text](images/configuration.png)


- Finally we disable the default host thats listening on port 80, to then create a html file for your site. A reload of nginx is required to kick start everything. The commands can be seen below.

```
sudo unlink /etc/nginx/sites-enabled/default
sudo systemctl reload nginx
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

![alt text](images/unlink_restart.png)

![alt text](images/new_web.png)


### Integrate php with nginx

- Next we are going to test whether nginx can give .php files to the php processor.
- I will run `sudo nano /var/www/projectLEMP/info.php` to create a php file which i will then place in
```
<?php
phpinfo();
```
which in turn should give back server information to be displayed in the browser when i enter 'http://ec2_public_ip//info.php'. The result for this can be seen below.

![alt](images/php_info_web.png)

- This is sensitive information about the server so i deleted the info.php file using `sudo rm /var/www/projectLEMP/info.php`.



### Retrieving data from MySQL with PHP

- Next I move on to creating test data, namely a to-do list in my database, to then configure it so that the info from the DB is displayed on my nginx website.
- Firstly I enetered the DB through `sudo mysql`, I then created a database using `CREATE DATABASE 'example_database';`
- Secondly I created an example user to which I'd give permissions to the just created database. `CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`, `GRANT ALL ON example_database.* TO 'example_user'@'%';`.


![alt text](images/db_creation.png)


- Small test to see if user creation was successuful via `mysql -u example_user -p`, along with `SHOW DATABASES;` to see the database.


![alt text](images/mysql_user_test.png)


- Next we move on to creating a test table and inserting a few rows.

```
# creating the table

CREATE TABLE example_database.todo_list (
item_id INT AUTO_INCREMENT,
content VARCHAR(255),
PRIMARY KEY(item_id)
);

```

` INSERT INTO example_database.todo_list (content) VALUES ("MY VALUES");`

![alt text](images/mysql_user_test.png)


- Finally we have to move on to configuring a script so that our php can connect to the database. This was done with the following script that was placed in '/var/www/projectLEMP' named todo_list.php.

```
<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";

try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}

```

The final result it the image below.

![alt](images/final_result.png)
