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
 * [CAP theorem](#cap-theorem)
 * [Domain name system](#domain-name-system)
 * [Content delivery network](#content-delivery-network)

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

#### Cached Database Queries

>That’s still the most commonly used caching pattern. Whenever you do a query to your database, you store the result dataset in cache. A hashed version of your query is the cache key. The next time you run the query, you first check if it is already in the cache. The next time you run the query, you check at first the cache if there is already a result. This pattern has several issues. The main issue is the expiration. It is hard to delete a cached result when you cache a complex query (who has not?). When one piece of data changes (for example a table cell) you need to delete all cached queries who may include that table cell. You get the point?

### Load balancing

>In computing, **load balancing** refers to the process of distributing a set of tasks over a set of resources (computing units), with the aim of making their overall processing more efficient. Load balancing techniques can optimize the response time for each task, avoiding unevenly overloading compute nodes while other compute nodes are left idle.
>Load balancers can route traffic based on various metrics, including:
        
- Random
- Least loaded
- Session/cookies
- Round robin or weighted round robin
- Layer 4
- Layer 7

[more](https://en.wikipedia.org/wiki/Load_balancing_(computing))

### Database replication

>Database replication can be used on many database management systems (DBMS), usually with a master-slave relationship between the original and the copies. The master logs the updates, which then ripple through to the slaves. Each slave outputs a message stating that it has received the update successfully, thus allowing the sending of subsequent updates.
    
#### Master-slave replication

>The master serves reads and writes, replicating writes to one or more slaves, which serve only reads. Slaves can also replicate to additional slaves in a tree-like fashion. If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned.

>Database replication becomes more complex when it scales up horizontally and vertically. Horizontal scale-up has more data replicas, while vertical scale-up has data replicas located at greater physical distances. Problems raised by horizontal scale-up can be alleviated by a multi-layer, multi-view access protocol. The early problems of vertical scale-up have largely been addressed by improving Internet reliability and performance.

#### Master-master replication

>Both masters serve reads and writes and coordinate with each other on writes. If either master goes down, the system can continue to operate with both reads and writes.

#### Federation

>Federation (or functional partitioning) splits up databases by function. For example, instead of a single, monolithic database, you could have three databases: forums, users, and products, resulting in less read and write traffic to each database and therefore less replication lag. Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality. With no single central master serializing writes you can write in parallel, increasing throughput.

#### Sharding

>Sharding distributes data across different databases such that each database can only manage a subset of the data. Taking a users database as an example, as the number of users increases, more shards are added to the cluster.
 Similar to the advantages of federation, sharding results in less read and write traffic, less replication, and more cache hits. Index size is also reduced, which generally improves performance with faster queries. If one shard goes down, the other shards are still operational, although you'll want to add some form of replication to avoid data loss. Like federation, there is no single central master serializing writes, allowing you to write in parallel with increased throughput.
 Common ways to shard a table of users is either through the user's last name initial or the user's geographic location.

### Database partitioning
      
>A partition is a division of a logical database or its constituent elements into distinct independent parts. Database partitioning is normally done for manageability, performance or availability[1] reasons, or for load balancing. It is popular in distributed database management systems, where each partition may be spread over multiple nodes, with users at the node performing local transactions on the partition. This increases performance for sites that have regular transactions involving certain views of data, whilst maintaining availability and security.
   
[more](https://en.wikipedia.org/wiki/Partition_(database))
eg: *ElasticSearch*

### CAP theorem

>Dr. Eric Brewer gave a keynote speech at the Principles of Distributed Computing conference in 2000 called 'Towards Robust Distributed Systems' [1]. In it he posed his 'CAP Theorem' - at the time unproven - which illustrated the tensions between being correct and being always available in distributed systems.
 Two years later, Seth Gilbert and Professor Nancy Lynch - researchers in distributed systems at MIT - formalised and proved the conjecture in their paper “Brewer's conjecture and the feasibility of consistent, available, partition-tolerant web services” [2].

  * Consistency - Every read receives the most recent write or an error
  * Availability - Every request receives a response, without guarantee that it contains the most recent version of the information
  * Partition Tolerance - The system continues to operate despite arbitrary partitioning due to network failures

#### CP - consistency and partition tolerance
>Waiting for a response from the partitioned node might result in a timeout error. CP is a good choice if your business needs require atomic reads and writes.

#### AP - availability and partition tolerance
>Responses return the most readily available version of the data available on any node, which might not be the latest. Writes might take some time to propagate when the partition is resolved.
AP is a good choice if the business needs allow for eventual consistency or when the system needs to continue working despite external errors.
    
### Domain name system

>DNS is hierarchical, with a few authoritative servers at the top level. Your router or ISP provides information about which DNS server(s) to contact when doing a lookup. Lower level DNS servers cache mappings, which could become stale due to DNS propagation delays. DNS results can also be cached by your browser or OS for a certain period of time, determined by the time to live (TTL).
    
[more](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd197427(v=ws.10)?redirectedfrom=MSDN)
    
### Content delivery network

>A CDN helps to speed up static components of your blog (especially pictures of say, your bed in Rome) by distributing them across a number of servers around the world. This does two things – one is to put much of your travel blog’s content closer to the person who wants to view it. So, a person in Japan will download your beautiful photos of Granada from Tokyo, rather than your server sitting in Boston, for example. Less distance traveled means a faster download and secondly, if the content isn’t being pulled directly from your server, it saves overall computing power (important to have plenty of for when traffic gets really busy).    
    
Serving content from CDNs can significantly improve performance in two ways:
    - Users receive content from data centers close to them    
    - Your servers do not have to serve requests that the CDN fulfills

### Microservices

>Related to this discussion are microservices, which can be described as a suite of independently deployable, small, modular services. Each service runs a unique process and communicates through a well-defined, lightweight mechanism to serve a business goal.
    

### Materials

 - [Paxos算法](https://zh.wikipedia.org/wiki/Paxos%E7%AE%97%E6%B3%95)
 - [Database-Replication](https://en.wikipedia.org/wiki/Replication_(computing)#DATABASE)
 - [Raft](https://www.usenix.org/system/files/conference/atc14/atc14-paper-ongaro.pdf)
 - [拜占庭将军问题](https://zh.wikipedia.org/wiki/%E6%8B%9C%E5%8D%A0%E5%BA%AD%E5%B0%86%E5%86%9B%E9%97%AE%E9%A2%98)