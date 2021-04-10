MongoDB Cluster
===
Building MongoDB for record all UiPath robot execution logs.

## Table of Contents

[TOC]

## Hardware info

Servers:
* QuantaGrid D52Y-2U : 3 
    * CPU: 6 * Intel® Xeon® Gold 5218 
    * RAM: 36 * Skhynix 2666Hz 2R*4 32G
    * Boot: 3 * Intel® SSD D3-S4610 240G
    * SSD: 24 * Intel® SSD D3-S4610 1.92TB
    * NIC: 3 * Broadcom® BCM57414 25G(SFP28)
    * SAS Controller: 3 * Broadcom® LSI 3316(HW RAID)

## Software info

| OS | Application |
| -------- | -------- |
| RHEL 7.6     | MongoDB Community 4.4.0    |

## Host info

|        | Server1<br />10.106.25.113 | Server2<br />10.106.25.114 | Server3<br />10.106.25.115 |
| ------ |:-------------------------- |:-------------------------- |:-------------------------- |
| Shard1 | Primary<br />Port: 20001   | Secondary<br />Port: 20001 | Arbiter<br />Port: 20001   |
| Shard2 | Arbiter<br />Port: 20002   | Primary<br />Port: 20002   | Secondary<br />Port: 20002 |
| Shard3 | Secondary<br />Port: 20003 | Arbiter<br />Port: 20003   | Primary<br />Port: 20003   |
| Config | Config<br />Port: 30000    | Config<br />Port: 30000    | Config<br />Port: 30000    |
| Route  | Route<br />Port: 40000     | Route<br />Port: 40000     | Route<br />Port: 40000     |


## Deployment




###### tags: `MongoDB` `Documentation`
