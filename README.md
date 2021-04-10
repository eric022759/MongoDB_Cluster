MongoDB Cluster
===
Building MongoDB for record all UiPath robot execution logs.

## Table of Contents

[TOC]

## Hardware info

| Server             | Hardware Spec                 | Amount |
|:------------------ |:----------------------------- |:------ |
| QuantaGrid D52Y-2U | Storage SKU                   | 3      |
| CPU                | Intel® Xeon® Gold 5218        | 6      |
| RAM                | Skhynix 2666Hz 2R4 32G        | 36     |
| Boot               | Intel® SSD D3-S4610 240G      | 3      |
| SSD                | Intel® SSD D3-S4610 1.92TB    | 24     |
| NIC                | Broadcom® BCM57414 25G(SFP28) | 3      |
| SAS Controller     | Broadcom® LSI 3316(HW RAID)   | 3      |


## Software info

| OS | Application |
| -------- | -------- |
| RHEL 7.6     | MongoDB Community 4.4.0    |

## Host info

|        | Server1<br/>10.106.25.113 | Server2<br/>10.106.25.114 | Server3<br/>10.106.25.115 |
| ------ |:-------------------------- |:-------------------------- |:-------------------------- |
| Shard1 | Primary<br/>Port: 20001   | Secondary<br/>Port: 20001 | Arbiter<br/>Port: 20001   |
| Shard2 | Arbiter<br/>Port: 20002   | Primary<br/>Port: 20002   | Secondary<br/>Port: 20002 |
| Shard3 | Secondary<br/>Port: 20003 | Arbiter<br/>Port: 20003   | Primary<br/>Port: 20003   |
| Config | Config<br/>Port: 30000    | Config<br/>Port: 30000    | Config<br/>Port: 30000    |
| Route  | Route<br/>Port: 40000     | Route<br/>Port: 40000     | Route<br/>Port: 40000     |


## Deployment
Authorize RHEL:

```
subscription-manager register

#login account/password

subscription-manager attach --auto

yum update
```

Install MongoDB:

* official document:<br/> https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/
* Mongod:<br/> 
https://docs.mongodb.com/manual/reference/program/mongod/
* Mongos:<br/>
https://docs.mongodb.com/manual/reference/program/mongos/


Create a /etc/yum.repos.d/mongodb-org-4.4.repo file
```
vi /etc/yum.repos.d/mongodb-org-4.4.repo
```
enter these lines
```
[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```
install with yum
```
yum install mongodb-org
```







###### tags: `MongoDB` `Documentation`
