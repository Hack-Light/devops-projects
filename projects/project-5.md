# PROJECT 5 - CLIENT/SERVER ARCHITECTURE USING A MYSQL RELATIONAL DATABASE MANAGEMENT SYSTEM

![project5](../images/project5/1.png)

> On the server `mysql server` Don't turn off remote access while installing mysql.

### SETUP MYSQL SERVER

> The steps to take are outlined in Project 1 and 2

![project5](../images/project5/2.png)

![project5](../images/project5/3.png)

![project5](../images/project5/5.png)

![project5](../images/project5/6.png)

![project5](../images/project5/7.png)

### SETUP MYSQL CLIENT

On the `client server` run

```bash
sudo apt update
sudo apt-get install mysql-client
```

![project5](../images/project5/8.png)

### ACTUAL CONFIGURATION

> By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client.
> MySQL server uses `TCP port 3306` by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in `mysql server` Security Groups. For extra security, do not allow all IP addresses to reach your `mysql server` – allow access only to the specific local IP address of your ‘mysql client’.

- You might need to configure MySQL server to allow connections from remote hosts.

```bash
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

- Replace ‘127.0.0.1’ to ‘0.0.0.0’ like this:

![project5](../images/project5/11.png)

![project5](../images/project5/9.png)

![project5](../images/project5/10.png)

- Restart the server

`sudo systemctl restart mysql.service`

![project5](../images/project5/12.png)

- Create Remote MySQL user and grant remote access to databases

```mysql
CREATE USER 'username'@'%' IDENTIFIED BY 'new-password';
FLUSH PRIVILEGES;
```

- Then you can grant access to databases using the `GRANT ALL` command: \

`GRANT ALL PRIVILEGES ON dbname.* TO 'username'@'%';`

- If you want to grant access to all databases on the server, run:\
  `GRANT ALL PRIVILEGES ON *.* TO 'user'@'%';`

- Once connected run\
  `Show databases;`

- On the mysql client run the command `mysql -h <host> -u <username> -p`
  ![project5](../images/project5/14.png)

  ![project5](../images/project5/15.png)

  ![project5](../images/project5/16.png)
