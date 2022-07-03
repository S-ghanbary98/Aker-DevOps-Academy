# Client/Server Architecture Using MySQL


## Intro

 In this project I'm going to build a client-server architecture which is a concept where two computers are connected together over a network to send and recieve request between one another. The two machines being client (sending the request) and server (responding to the requests). This will be done using SQL

 ![client/server diagram](img/architecture.png)

 MySQL is an open-source relational database management system.

## EC2 Configuration

- Firstly two ubuntu 22.04 ec2's are lauched, one the client the other the server. 

 ![ec2](img/ec2.png)

 Firstly both ec2 were updated and mysql was installed via the following command below. We start of with our client server. 

 ```
sudo apt update && sudo apt upgrade -y
sudo apt-get install mysql-server

systemctl is-active mysql  # verify

 ```

 ![verify_myqsl](img/verify_mysql.png)
 ![verify_mys](img/active_mysql.png)

 - Byt default both of our ec2 are in the same local virtual network, MySQL server uses TCP port 3306 which is configured in mysql-server ibound security group.
 - Mysql-server is configured to allow connections from mysql-client, via `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`, the bind address is changed to 0.0.0.0 to allow all. Afterwhich the client server is restarted through `sudo systemctl restart mysql`

 ![server-config](img/bind-address.png)

## User Creation 

- In the mysql-server instance we log in to mysql anc create users and grant those users remote access to our instance.

```
sudo mysql
GRANT ALL ON database_name.* TO user_name@'ip_address' IDENTIFIED BY 'user_password';

```
 ![user](img/users.png)

 ## Remote Connection

 - Finally in our mysql-server instance we allow access to the IP of the mysql-client.

 ```
 sudo ufw allow from <mysql-client port> to any port 3306

# or allow all IP's
sudo ufw allow 3306

 ```

  ![port](img/allow_port.png)

  - We can now connect to our mysql-server through our mysql-client via

  ```
  mysql -u user_name -h mysql_server_ip -p

  ```


  ![connection](img/connection.png)
