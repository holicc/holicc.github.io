---
title: Common Principles
date: 2020-10-10
categories:
  - system-design
markup: mmark
math: true
tags:
    - system-design
---

## Contents
 * [Vertical scaling](#vertical-scaling)
 * [Horizontal scaling](#horizontal-scaling)
 * [Caching](#caching)
 * [Load balancing](#load-balancing)
 * [Database replication](#database-replication)
 * [Database partitioning](#database-partitioning)

### Vertical scaling

**Vertical scaling** means add more new resources in the existing system to meet the expectation.
In other words that you scale by adding more power (CPU, RAM) to an existing machine.

Good example of vertical scaling is *The cloud version of MySQL*

### Horizontal scaling

**Horizontal scaling** means that you scale by adding more machines.
When added new resources to the existing system can't meet the expectation,we need to add completely new servers 
and distraction our services to these machines.

Good examples of horizontal scaling are *Cassandra, MongoDB*

### Caching

**Caching** means temporary storage location,usually store in RAM.So that they can be accessed more quickly.  

### Load balancing

    In computing, **load balancing** refers to the process of distributing a set of tasks over a set of resources (computing units), with the aim of making their overall processing more efficient. Load balancing techniques can optimize the response time for each task, avoiding unevenly overloading compute nodes while other compute nodes are left idle.

[more](https://en.wikipedia.org/wiki/Load_balancing_(computing))

### Database replication

    Database replication can be used on many database management systems (DBMS), usually with a master-slave relationship between the original and the copies. The master logs the updates, which then ripple through to the slaves. Each slave outputs a message stating that it has received the update successfully, thus allowing the sending of subsequent updates.
    
[more](https://en.wikipedia.org/wiki/Replication_(computing)#DATABASE)

keywords: *master-slave relationship*,*multi-master replication*,*distributed transaction*

    Database replication becomes more complex when it scales up horizontally and vertically. Horizontal scale-up has more data replicas, while vertical scale-up has data replicas located at greater physical distances. Problems raised by horizontal scale-up can be alleviated by a multi-layer, multi-view access protocol. The early problems of vertical scale-up have largely been addressed by improving Internet reliability and performance.

### Database partitioning
      
    A partition is a division of a logical database or its constituent elements into distinct independent parts. Database partitioning is normally done for manageability, performance or availability[1] reasons, or for load balancing. It is popular in distributed database management systems, where each partition may be spread over multiple nodes, with users at the node performing local transactions on the partition. This increases performance for sites that have regular transactions involving certain views of data, whilst maintaining availability and security.
   
[more](https://en.wikipedia.org/wiki/Partition_(database))
eg: *ElasticSearch*


