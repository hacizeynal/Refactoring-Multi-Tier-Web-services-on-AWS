# Refactoring services on AWS

## Introduction

In this project we will  refactor services for AWS from previous project ,for more info please check previous [project](https://github.com/hacizeynal/Multi-Tier-Web-App-Setup-on-AWS).The aim of this  approach is to boost agility or improve business continuity.

**PROBLEM**

* Operational overhead <br>
* Struggling for the uptime and regular scaling requirement <br>
* Upfront CapEx & Regular OpEx <br>
* Manual processing /Difficult to Automate <br>

**SOLUTION**

We can use Cloud services ,instead of using IAAS we will be using PAAS & SAAS ,it means we will not use regular EC2 instances ,we will use managed services. Compared to IAAS services ,it is very easy to manage,control and scale those applications.

Following AWS services will be used in this project for Frontend

* Beanstalk >> VM for Tomcat Application server 
* Beanstalk >> Automation for VM Scaling 
* Beanstalk >> NGINX Load Balancer Replacement 
* S3/EFS >> Storage 

Following AWS services will be used in this project for Backend

* RDS Instance >> Databases 
* Elasticcache >> Memcached 
* Active MW >> Rabbit MQ 
* Route53 >> DNS 
* CloudFront >> Content Delivery Network
* AWS CloudWatch >> Monitoring

**OBJECTIVE**

* Flexible Infra 
* No Upfront Cost 
* IAAC 
* PAAS & SAAS for low operational overhead

**High level of traffic flow is described below** 

[![Screenshot-2022-10-21-at-11-36-01.png](https://i.postimg.cc/xCVKdmvD/Screenshot-2022-10-21-at-11-36-01.png)](https://postimg.cc/Y484nhDb)

**Flow of Execution**

[![Screenshot-2022-10-21-at-11-40-08.png](https://i.postimg.cc/qBbxsN0n/Screenshot-2022-10-21-at-11-40-08.png)](https://postimg.cc/jCffKdDd)

[![Screenshot-2022-10-21-at-11-41-04.png](https://i.postimg.cc/j5ChWf2M/Screenshot-2022-10-21-at-11-41-04.png)](https://postimg.cc/S2F93JW9)

--------------------------------------------------------------------------------------------------------------------
## Configure RDS

We will start configuration from Backend services.

Since we have already created Security Groups in previous project ,we will re-use them again ,because all allowed ports will be same.We will create parameter and subnet group and attach them to RDS configuration.
Next step will be creating RDS on AWS.

* Engine version >> MySQL 5.7.39
* Instance class >> db.t3.micro (Free Tier)
* Multi-AZ enabled >> No
* VPC/Subnet >> Dedicated VPC for DevOps projects

Please make sure to save credentials if you choose auto-password generate option.

[![Screenshot-2022-10-21-at-12-25-27.png](https://i.postimg.cc/1XgThtqj/Screenshot-2022-10-21-at-12-25-27.png)](https://postimg.cc/zHrpTqpC)

## Configure Elasticache

As RDS ,we will also create parameter group and subnet group and attach them to Elasticache configuration.

* Engine >> Memcache
* Node Type >> cache.t3.micro
* Engine version >> 1.4.5
* VPC/Subnet >> Dedicated VPC for DevOps projects

[![Screenshot-2022-10-21-at-12-40-01.png](https://i.postimg.cc/6qrHN10D/Screenshot-2022-10-21-at-12-40-01.png)](https://postimg.cc/xNdGv5BR)

## Configure Amazon MQ

* Broker instance type >> mq.t3.micro
* Broker engine >> RabbitMQ
* Broker engine version >> 3.9.16

[![Screenshot-2022-10-21-at-12-49-49.png](https://i.postimg.cc/kgRP07SZ/Screenshot-2022-10-21-at-12-49-49.png)](https://postimg.cc/pmvS8bqY)

## Configure MySQL Client

We will create one more EC2 instance to manage MySQL Database and we will login to RDS in order to initialize database.

Let's check if we can connect to MySQL from our client.

```
mysql -h rds-mysql.cvqlmbpezuq0.us-east-1.rds.amazonaws.com -u admin -pOEHlJvDdt7uoBC4Z9lB

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| accounts           |
| innodb             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)

mysql>

root@ip-172-18-172-160:~# git clone https://github.com/hacizeynal/Refactoring-services-on-AWS.git

cd src/main/

root@ip-172-18-172-160:~/Refactoring-services-on-AWS/src/main/resources# ls -la

drwxr-xr-x 2 root root 4096 Oct 22 08:59 .
drwxr-xr-x 5 root root 4096 Oct 22 08:59 ..
-rw-r--r-- 1 root root  719 Oct 22 08:59 application.properties
-rw-r--r-- 1 root root 6176 Oct 22 08:59 db_backup.sql
-rw-r--r-- 1 root root  581 Oct 22 08:59 logback.xml
-rw-r--r-- 1 root root  280 Oct 22 08:59 validation.properties

// Initilize DB with schema 

mysql -h rds-mysql.cvqlmbpezuq0.us-east-1.rds.amazonaws.com -u admin -pOEHlJvDdt7uoBC4Z9lBC accounts < db_backup.sql 

root@ip-172-18-172-160:~/Refactoring-services-on-AWS/src/main/resources# mysql -h rds-mysql.cvqlmbpezuq0.us-east-1.rds.amazonaws.com -u admin -pOEHlJvDdt7uoBC4Z9lBC accounts
mysql: [Warning] Using a password on the command line interface can be insecure.
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 271
Server version: 5.7.39 Source distribution

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show tables;
+--------------------+
| Tables_in_accounts |
+--------------------+
| role               |
| user               |
| user_role          |
+--------------------+
3 rows in set (0.00 sec)

mysql> 

```
## Configure Elastic Beanstalk

Beanstalk will automate following services and help to deploy our application.

* EC2 instances (depends on if we will use LoadBalancer)
* Elastic Load Balancer (default is ALB)
* Autoscaling Group for EC2
* Security/VPC configuuration and etc

We can think Application as a big container and within the Application, we can create multiple different environments with different configurations.

We can validate Beanstalk via clicking URL on default Application ,it should give us Welcome Page.

[![Screenshot-2022-10-22-at-21-22-24.png](https://i.postimg.cc/d3s4sM2F/Screenshot-2022-10-22-at-21-22-24.png)](https://postimg.cc/jw1zcF33)

Please make sure to update SG if needed with needed ports in order to allow traffic sourced from EC2 (created via Beanstalk) to Backend services. We need also to add second listener with port 443 with SSL certificate in our Load Balancer configuration.







