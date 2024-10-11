# My Learning Experience: Establishing Client-Server Architecture with MySQL on AWS EC2

## This README outlines my experience setting up a client-server architecture with MySQL using two AWS EC2 instances. I will walk through the steps I followed, the challenges I faced, and the lessons I learned during the process.

# Table of Contents
+ Introduction
+ Prerequisites
+ Architecture Overview
+ Setting Up MySQL Server (mysql-server instance)
+ Installation
+ Initial Configuration
+ Troubleshooting: Regaining Root Access
+ Configuring MySQL for Remote Access
+ Setting Up MySQL Client (mysql-client instance)
+ AWS Security Group Configuration
+ Network Diagnostics: Ping and Traceroute
+ Final Steps and Reflections
+ Additional Notes

  
### Introduction  
This guide captures my experience setting up a client-server architecture with MySQL on two AWS EC2 instances. Along with the step-by-step process, I'll highlight the challenges I encountered and the insights I gained. Whether you're new or experienced, I hope my journey provides valuable guidance for your own setup.  

### Prerequisites  
Before starting the project, I ensured the following:  
- Two AWS EC2 instances (named `mysql-server` and `mysql-client`) running Ubuntu.  
- A basic understanding of AWS security groups, Linux commands, and MySQL operations.  
- SSH access to both instances.  
![image](https://github.com/user-attachments/assets/e9f9a034-61e5-495c-ac62-8544371629d5)


# Architecture Overview
My setup consists of:

+mysql-client: MySQL Client instance
+mysql-server: MySQL Server instance
Both instances are in the same VPC for better security and performance.
![image](https://github.com/user-attachments/assets/017ded95-0b92-4999-8171-e8bec19964d0)

# Setting Up MySQL Server (mysql-server instance)

## Installation 
+ i SHH into the mysql-server instance
+ I uploaded package list and installed MySQL server

```
sudo apt update
sudo apt install mysql-server -y
```
## Initial Configuration
+ Started and enabled the MySQL service:
```
sudo systemctl start mysql
sudo systemctl enable mysql
```
![image](https://github.com/user-attachments/assets/9be505ad-96f8-43c1-b9d9-1739f6fc38ec)

## Set the root password:
```
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_strong_password';
FLUSH PRIVILEGES;
EXIT;
```
![image](https://github.com/user-attachments/assets/2dd21911-9263-4046-b6ce-677b49f0685c)
*in this case i used 'your_strong_password' as my password*

  
## Ran the secure installation script:
```
sudo mysql_secure_installation
```
![image](https://github.com/user-attachments/assets/ca18807b-0f33-4341-b603-9d24ed639a17)

*Note: I made the mistake of running the `mysql_secure_installation` script before setting the root* *password. This resulted in being locked out of the root account, teaching me the crucial lesson of adhering to the correct procedural order.*

## Verified the installation:
```
sudo systemctl status mysql
```
![image](https://github.com/user-attachments/assets/4a2ea2c5-8a36-4308-80ec-3e69ec6718e7)



*pls not that it is important to set the root password before running  mysql_secure_installation if you do otherwise*  *and oyou mistakenly logged your self out of the root account you did need to follow some proceedure to regain access*

## Configuring MySQL for Remote Access
+ Opened the MySQL configuration file:
 ```
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
 ```
+ Identified and modified two bind-address settings:
```
bind-address = 0.0.0.0
mysqlx-bind-address = 127.0.0.1
```
![image](https://github.com/user-attachments/assets/877e473e-36b2-458c-934c-6f55b55ce888)

*Personal Note*: I initially found two `bind-address` settings in the configuration file, which caused some confusion. Here's what I learned about each:

- **bind-address**: This setting determines which IP address MySQL listens to for standard client connections (usually on port 3306). Setting it to `0.0.0.0` allows connections from any IP address.
- **mysqlx-bind-address**: This setting is for the MySQL X Protocol, which supports document-based CRUD operations and JSON interactions, typically on port 33060. Since I wasn't using X Protocol features, I left this at the default value of `127.0.0.1`.

+ Restarted MySQL:
```
sudo systemctl restart mysql
```
+ Created a MySQL user for remote access:

i log in into mysql using 
```
sudo mysql -u root -p
```
then i created a user using the code below 
```
CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Password.1';
GRANT ALL PRIVILEGES ON *.* TO 'remote_user'@'%';
FLUSH PRIVILEGES;
```
![image](https://github.com/user-attachments/assets/fad033ed-8248-4bbb-b305-500c74006b13)




# Setting Up MySQL Client (mysql-client instance)

+SSH'd into the mysql-client instance.
+Installed MySQL client:
```
sudo apt update
sudo apt install mysql-client -y
```
+Verified the installation:
```
mysql --version
```
*Note The setup went smoothly, but make sure both instances are on the same VPC for improved* *connectivity and security.*

## AWS Security Group Configuration

Rather than configuring a firewall directly on the instance, I chose to handle traffic using AWS Security Groups. Here's the setup I used:

+ Edited the inbound rules for the security group associated with mysql-server to allow MySQL traffic from mysql-client's private IP.
+ Rule configuration
```
Type: MySQL/Aurora
Protocol: TCP
Port Range: 3306
Source: <mysql-client Private IP>/32
```
+ mysql-client: Ensured mysql-client had the necessary rules for SSH access.
  
*Note: Handling security at the AWS level provides an additional layer of control, enabling more detailed access* *management without needing to alter the server itself.*


 

# Establishing Connection

Once the security groups were configured, I connected the MySQL client from mysql-client to the MySQL server on mysql-server.

+ From mysql-client, I used the following command to connect:
  
```
mysql -h <mysql-server Private IP> -u remote_user -p
```
+ Entered the password when prompted and successfully connected to the MySQL server running on mysql-server.

  ![image](https://github.com/user-attachments/assets/5ce23081-0744-4a8e-8ef8-c4b36322aeba)


*Note: This step was crucial to the success of my setup. Properly configuring the security group was essential for everything to function correctly. Also, i made a mistake of entering the wrong password this was a set back to me but i qickly realise that the password was wrong and made the corrections.*



 

## Network Diagnostics: Ping and Traceroute

+  Ping
To test the network connectivity between the instances, I used the ping command:
```
ping <mysql-server Private IP>
```
![image](https://github.com/user-attachments/assets/8b1d5851-ed66-425b-bd9a-7d935ddd26a8)


+ Traceroute
For a detailed path analysis, I used traceroute:
```
traceroute <mysql-server Private IP>
```
![image](https://github.com/user-attachments/assets/54264467-9a69-4fab-ae03-e6df85b1dfba)

*Personal Note: Both ping and traceroute were useful in troubleshooting connectivity problems, confirming that the* * mysql-client could connect with the mysql-server on the internal network *







 # Final Steps and Reflections

 ## Final Checklist
 
+ Database Operations: After connecting to the MySQL server, I could seamlessly create databases, tables, and execute queries.

Here's an example of a simple query to create a database:

```
CREATE DATABASE my_database;
```

+ Security: I ensured that the security group rules were correctly configured and only allowed traffic between the two instances.



![image](https://github.com/user-attachments/assets/9f90b0b5-898b-4ecc-85c9-b56fb09a1a7a)
 


# My Personal Experience and what i have learnt so far 
This experience emphasized the crucial role AWS security groups play in managing instance connectivity. One of my key takeaways was deepening my understanding of how MySQL operates within a client-server architecture. Using AWS security groups, rather than configuring firewalls directly on the instance, felt more scalable and manageable, especially in cloud environments.

## Challenges i Faced
At first, I encountered connectivity problems due to misconfigured security groups, but careful adjustments solved the issue. Learning to configure MySQL for remote access was vital, and understanding the potential security risks helped me secure the setup effectively. Accidentally locking the root account during setup also taught me the importance of following the correct sequence of steps when configuring MySQL.

## Additional Notes
**Running MySQL Instances Across Different Regions or Cloud Providers**
Although my setup was within the same AWS region, I discovered that it’s possible to run MySQL server and client instances across different regions or cloud environments. Here are key considerations:

- **Latency:** Instances in different regions may face higher latency, so selecting regions close to each other is important.
  
- **Networking:** Ensure proper networking through options like VPN or VPC peering. For cross-cloud environments, consider services like AWS Direct Connect or similar.

- **Security:** Pay extra attention to firewall rules and security groups, and consider using SSL/TLS for secure, encrypted connections.

- **Costs:** Be mindful of potential data transfer costs between regions or across cloud providers.
