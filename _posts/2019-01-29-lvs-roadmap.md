---
layout: post
title:  "Linux Virtual Server 学习路线图"
date:   2019-01-29 11:18:23 +0800
tags: [LVS]
---

官网：<http://www.linux-vs.org/>

### What is virtual server?

<http://www.linux-vs.org/whatis.html>

Virtual server is a highly scalable and highly available server built on a cluster of real servers. The architecture of server cluster is fully transparent to end users, and the users interact with the cluster system as if it were only a single high-performance virtual server.
￼

### Why virtual server?

<http://www.linux-vs.org/why.html> 

### How virtual server works?

<http://www.linux-vs.org/how.html>
* Virtual Server via NAT
* Virtual Server via IP Tunneling
* Virtual Server via Direct Routing

### General Architecture of LVS Clusters

<http://www.linux-vs.org/architecture.html> 

### High Availability

<http://www.linux-vs.org/HighAvailability.html> 

* Using Piranha to build highly available LVS systems
* Using Keepalived to build highly available LVS systems
* Using UltraMonkey to build highly available LVS systems
* Using heartbeat+mon+coda to build highly available LVS systems
* Using heartbeat+ldirectord to build highly available LVS systems

