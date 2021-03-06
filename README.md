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
**Note: Highly recommending build over 3 config servers in different rack of server.**

This practice only build 3 vm in one server.

|        | Server1<br/>10.106.25.113 | Server2<br/>10.106.25.114 | Server3<br/>10.106.25.115 |
| ------ |:-------------------------- |:-------------------------- |:-------------------------- |
| Shard1 | **Primary**<br/>Port: 20001   | Secondary<br/>Port: 20001 | Arbiter<br/>Port: 20001   |
| Shard2 | Arbiter<br/>Port: 20002   | **Primary**<br/>Port: 20002   | Secondary<br/>Port: 20002 |
| Shard3 | Secondary<br/>Port: 20003 | Arbiter<br/>Port: 20003   | **Primary**<br/>Port: 20003   |
| Config | Config (**Primary**)<br/>Port: 30000    | Config (Secondary)<br/>Port: 30000    | Config (Secondary)<br/>Port: 30000    |
| Route  | Route<br/>Port: 40000     | Route<br/>Port: 40000     | Route<br/>Port: 40000     |

## Overall execute sequence:
Create mongodb-keyfile<br/>
Copy keyfile **to all server**<br/>
Change keyfile permission(chmod) & owner(chown) **in all server**<br/>
↓<br/>
Annotate .conf file → **Security.keyFile** setting<br/>
Start Shard1, Shard2, Shard3, Config service<br/>
↓<br/>
**Finish 3 shards** & **Finish Config Server** = total 32 times<br/>
(1) mongo IP_Address:Shard1_Port (Primary)<br/>
(2) activate Shard1 replica set → rs.initiate(config)<br/>
(3) use admin & create root user<br/>
(4) disannotate .conf file → **Security.keyFile** setting<br/>
(5) restart 3-VM MongoDB-Shard1.service<br/>
(6) mongo IP_Address:Shard1_Port (Primary)<br/>
(7) use admin & db.auth()<br/>
(8) rs.initiate() → rs.status()<br/>
↓<br/>
**Finish Mongos Server** (1)~(3)<br/>
#Disannotate /etc/mongos.conf file → **Security.keyFile** setting<br/>
#Restart Mongos service<br/>
sh.addShard(shard1, shard2, shard3)<br/>
↓<br/> 
Create database & collection<br/>
Enable collection sharding

## RHEL authorization

``` bash=
subscription-manager register

#login account/password

subscription-manager attach --auto

yum update
```

## Install MongoDB package
Install MongoDB:

* official document:<br/> https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/
* Mongod:<br/> 
https://docs.mongodb.com/manual/reference/program/mongod/
* Mongos:<br/>
https://docs.mongodb.com/manual/reference/program/mongos/


Create a /etc/yum.repos.d/mongodb-org-4.4.repo file
``` bash=
vi /etc/yum.repos.d/mongodb-org-4.4.repo

[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```
install with yum
``` bash=
yum install mongodb-org
```

## Pre-setup (before running service)
1. Create folder for DB and log storage.
``` bash=
mkdir -p /home/mongod/db /home/mongod/log /home/mongod/log/mongos
mkdir -p /home/mongod/db/shard1 /home/mongod/db/shard2 /home/mongod/db/shard3 /home/mongod/db/config
```

2. Create 5 service & config file → refer to .service & .conf file
``` txt
# create /etc/systemd/system/MongoDB-Shard1.service  ->  /etc/shard1.conf
# create /etc/systemd/system/MongoDB-Shard2.service  ->  /etc/shard2.conf
# create /etc/systemd/system/MongoDB-Shard3.service  ->  /etc/shard3.conf
# create /etc/systemd/system/MongoDB-Config.service  ->  /etc/config.conf
# create /etc/systemd/system/MongoDB-Mongos.service  ->  /etc/mongos.conf
```
 - Copy files into different server
``` bash=
cd /etc/
scp shard1.conf shard2.conf shard3.conf config.conf mongos.conf root@ip_address:/etc/

cd /etc/systemd/system/
scp MongoDB-Shard1.service MongoDB-Shard2.service MongoDB-Shard3.service MongoDB-Config.service MongoDB-Mongos.service root@ip_address:/etc/systemd/system
```
3. Add user/group and change directory owner<br/>
➞ The reason why don't create mongos user is because of security.keyFile setting. keyFile allows only one user to have  read write permission(chmod 600). It doesn't allow group user to access keyFile for security reason.
``` bash=
# create mongod user & change directory owner
sudo chown -R mongod:mongod /home/mongod

#create mongos user & change directory owner
# groupadd mongos; useradd -r -g mongod -s /bin/false mongos
# sudo chown -R mongos:mongod /home/mongod/log/mongos
```

4. Change SElinux configuration:

 - Option 1:<br/>
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/changing-selinux-states-and-modes_using-selinux
``` bash=
sudo yum install checkpolicy
sestatus #check security status(current mode)
vi /etc/selinux/config
```

``` bash=
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#       enforcing - SELinux security policy is enforced.
#       permissive - SELinux prints warnings instead of enforcing.
#       disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
#       targeted - Targeted processes are protected,
#       mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

```bash=
reboot

getenforce # check current security mode
```

 - Option 2:
```  bash=
getenforce # check security current mode
setenforce 0 # change security current mode to Permissive
reboot
```

5. Check firewall setting<br/>
https://docs.mongodb.com/manual/tutorial/configure-linux-iptables-firewall/
```bash=
systemctl status firewalld
#option1
systemctl stop firewalld # stop firewall

#option2
iptables -L # check ip tables

# allows all incoming traffic from <ip-address> on port 27017, which allows the application server to connect to the mongod instance.
iptables -A INPUT -s 10.106.25.200 -p tcp --destination-port 27017 -m state --state NEW,ESTABLISHED -j ACCEPT
#allows outgoing traffic from the mongod to reach the application server.
iptables -A OUTPUT -d 10.106.25.113 -p tcp --source-port 40000 -m state --state ESTABLISHED -j ACCEPT
```

## Start service
1. using systemctl start service
``` bash=
systemctl start MongoDB-Shard1.service
systemctl start MongoDB-Shard2.service
systemctl start MongoDB-Shard3.service
systemctl start MongoDB-Config.service
```

- debug use command
``` bash=
sudo systemctl daemon-reload #.service file update & reload
systemctl restart .service #restart service

systemctl status .service #debug use
journalctl -xe #debug use
journalctl | grep mongo #check mongo related logs

ps -aux | grep mongo # check process
lsof -nPi | mongo # check network 
netstat -nao | grep LISTEN # check network
```

2. Important process:<br/>
**Note: Shard1, Shard2, Shard3, Config server's replica set needs to be done before start Mongos service.**<br/>

- Shard1: using 10.106.25.113 server connect mongoDB<br/>
```bash=
mongo 10.106.25.113:20001
```
```json=
config = {
    "_id": "shard1",
    "members": [
        {
            "_id": 0,
            "host": "10.106.25.113:20001"
        },
        {
            "_id": 1,
            "host": "10.106.25.114:20001"
        },
        {
            "_id": 2,
            "host": "10.106.25.115:20001",
            "arbiterOnly": 1
        }
    ]
}

rs.initiate(config)
rs.status()
db.isMaster()

新增Admin：(必須要給予roles權限)
use admin
db.createUser(
  {
    user: "",
    pwd: "",
    roles: [ { role: "root", db: "admin" } ]
  }
)
```

- Shard2: using 10.106.25.114 server connect mongoDB<br/>
```bash=
mongo 10.106.25.114:20002
```
```json=
config = {
    "_id": "shard2",
    "members": [
        {
            "_id": 0,
            "host": "10.106.25.113:20002",
            "arbiterOnly": 1
        },
        {
            "_id": 1,
            "host": "10.106.25.114:20002"
        },
        {
            "_id": 2,
            "host": "10.106.25.115:20002"
        }
    ]
}

rs.initiate(config)
rs.status()
db.isMaster()

新增Admin：(必須要給予roles權限)
use admin
db.createUser(
  {
    user: "r",
    pwd: "q",
    roles: [ { role: "root", db: "admin" } ]
  }
)
```

- Shard3: using 10.106.25.115 server connect mongoDB<br/>
```bash=
mongo 10.106.25.115:20003
```
```json=
config = {
    "_id": "shard3",
    "members": [
        {
            "_id": 0,
            "host": "10.106.25.113:20003"
        },
        {
            "_id": 1,
            "host": "10.106.25.114:20003",
            "arbiterOnly": 1
        },
        {
            "_id": 2,
            "host": "10.106.25.115:20003"
        }
    ]
}

rs.initiate(config)
rs.status()

新增Admin：(必須要給予roles權限)
use admin
db.createUser(
  {
    user: "r",
    pwd: "q",
    roles: [ { role: "root", db: "admin" } ]
  }
)
```

- Create Secure replicaset connection keyfile
```bash=
10.106.25.113 VM
# Create encrypted keyfile
openssl rand -base64 741 > /etc/mongodb-keyfile
# Copy file to other mongod instance
cd /etc
scp mongodb-keyfile root@10.106.25.114:/etc
scp mongodb-keyfile root@10.106.25.115:/etc

#change file permission -> 10.106.25.113, 10.106.25.114, 10.106.25.115
chmod 600 /etc/mongodb-keyfile
#change user permission
chown mongod.mongod /etc/mongodb-keyfile
-------------------------------------------------
10.106.25.114 VM
openssl rand -base64 741 > /etc/mongodb-keyfile
cd /etc
scp shard2-keyfile root@10.106.25.113:/etc
scp shard2-keyfile root@10.106.25.115:/etc
chmod 600 /etc/shard2-keyfile
chown mongod.mongod /etc/shard2-keyfile
```
- Config: using 10.106.25.113 server connect mongoDB<br/>
```bash=
mongo 10.106.25.113:30000
```
```json=
config = {
    "_id": "mongodb-configsvr",
    "version": 1,
    "term": 1,
    "protocolVersion": 1,
    "writeConcernMajorityJournalDefault": true,
    "configsvr": true,
    "members": [
        {
            "_id": 0,
            "host": "10.106.25.113:30000",
        },
        {
            "_id": 1,
            "host": "10.106.25.114:30000",
        },
        {
            "_id": 2,
            "host": "10.106.25.115:30000",
        }

    ],
    "settings": {
        "chainingAllowed": true,
        "heartbeatTimeoutSecs": 10,
        "electionTimeoutMillis": 10000,
        "catchUpTimeoutMillis": -1
    }
}

rs.initiate(config)
rs.status()

新增Admin：(必須要給予roles權限)
use admin
db.createUser(
  {
    user: "r",
    pwd: "q",
    roles: [ { role: "root", db: "admin" } ]
  }
)
```

3. Start mongos service & create root account
```bash=
systemctl start MongoDB-Mongos.service

mongo 10.106.25.113:40000

新增Admin：(必須要給予roles權限)
use admin
db.createUser(
  {
    user: "r",
    pwd: "q",
    roles: [ { role: "root", db: "admin" } ]
  }
)

systemctl restart MongoDB-Mongos.service
```

## Check started service
- Check process & network
```bash=
ps -aux | grep mongo    
lsof -nPi | grep mongo
```
- Restart service 
```bash=
#systemctl restart MongoDB-Shard1.service
#systemctl restart MongoDB-Shard2.service
#systemctl restart MongoDB-Shard3.service
#systemctl restart MongoDB-Config.service
#systemctl restart MongoDB-Mongos.service
```

## Add/Config sharding cluster in Mongos
```bash=
mongo 10.106.25.113:40000
use admin
```
- config sharding cluster
```bash=
#db.runCommand({addshard: "shard1/10.106.25.113:20001,10.106.25.114:20001"}); # 10.106.25.115 arbiter
sh.addShard("shard1/10.106.25.113:20001,10.106.25.114:20001")

#db.runCommand({addshard: "shard2/10.106.25.114:20002,10.106.25.115:20002"}); # 10.106.25.113 arbiter
sh.addShard("shard2/10.106.25.114:20002,10.106.25.115:20002")

#db.runCommand({addshard: "shard3/10.106.25.113:20003,10.106.25.115:20003"}); # 10.106.25.114 arbiter
sh.addShard("shard3/10.106.25.113:20003,10.106.25.115:20003")
```
- enable database/collection sharding
```bash=
use admin
#config db into sharding
db.runCommand({enablesharding:"db_name"})
#config collection into sharding
db.runCommand({shardcollection:"db_name.collection_name", key:{_id:1}})
```
- test sharding result
```bash=
#assuming we are using uipath.log ➞ db.collection
for (var i=0; i<=50000; i++) db.log.insert({startT:i, user:"Test"})

db.log.stats()

db.printShardingStatus()

db.log.getShardDistribution()
```

## Connection string
Note: 因為＠是特殊符號，所以要encoding成%40<br/>

mongodb://user_name:pwd@IP_address:Port



## Copy data to local machine -> Service file
```bash=
scp root@10.106.25.113:/etc/systemd/system/{MongoDB-Shard1,MongoDB-Shard2,MongoDB-Shard3,MongoDB-Config,MongoDB-Mongos}.service \
/Users/ericyang/Desktop/MongoDB_Cluster/service/10.106.25.113

scp root@10.106.25.114:/etc/systemd/system/{MongoDB-Shard1,MongoDB-Shard2,MongoDB-Shard3,MongoDB-Config,MongoDB-Mongos}.service \
/Users/ericyang/Desktop/MongoDB_Cluster/service/10.106.25.114

scp root@10.106.25.115:/etc/systemd/system/{MongoDB-Shard1,MongoDB-Shard2,MongoDB-Shard3,MongoDB-Config,MongoDB-Mongos}.service \
/Users/ericyang/Desktop/MongoDB_Cluster/service/10.106.25.115
```


## Copy data to local machine -> Config file
```bash=
scp root@10.106.25.113:/etc/{shard1,shard2,shard3,config,mongos}.conf 
/Users/ericyang/Desktop/MongoDB_Cluster/config/10.106.25.113

scp root@10.106.25.114:/etc/{shard1,shard2,shard3,config,mongos}.conf \
/Users/ericyang/Desktop/MongoDB_Cluster/config/10.106.25.114

scp root@10.106.25.115:/etc/{shard1,shard2,shard3,config,mongos}.conf \
/Users/ericyang/Desktop/MongoDB_Cluster/config/10.106.25.115
```


###### tags: `MongoDB` `Cluster` `Documentation`



