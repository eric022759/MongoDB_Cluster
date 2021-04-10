---
title: 'MongoDB'
disqus: hackmd
---

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


| Hostname | roles    | IP       | replica set name | port |
| Shard1 | -------- | -------- | ---------------- | ---- |
| Shard2 | -------- | -------- | 1                | 1    |
| Shard3 | Text     | Text     | 1                | 1    |



###### tags: `MongoDB` `Documentation`
