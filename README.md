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






