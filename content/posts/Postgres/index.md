---
title: Postgres VS MySQL
date: 2020-08-26
categories:
  - database
markup: mmark
math: true
tags:
    - database
---

## History

POSTGRES的实现始于 1986 年，现在被称为PostgreSQL的**对象-关系型**数据库管理系统是从加州大学伯克利分校写的POSTGRES软件包发展而来的。经过二十多年的发展，PostgreSQL是世界上可以获得的最先进的开源数据库。

## Advantage

Postgres is an **object-relational** database, while MySQL is a purely **relational** database

Postgres handles concurrency better than MySQL for multiple reasons:

 - Postgres implements Multiversion Concurrency Control (MVCC) without read locks(不使用锁来实现MVCC机制) 
 - Postgres supports parallel query plans that can use multiple CPUs/cores(多核利用)
 - Postgres can create indexes in a non-blocking way (through the CREATE INDEX CONCURRENTLY syntax), and it can create partial indexes (for example, if you have a model with soft deletes, you can create an index that ignores records marked as deleted) 
 - Postgres is known for protecting data integrity at the transaction level. This makes it less vulnerable to data corruption.(安全性)

Postgres is highly extensible. It supports a number of advanced data types not available in MySQL (geometric/GIS, network address types, JSONB which can be indexed, native UUID, timezone-aware timestamps). If this is not enough, **you can also add your own data-types, operators, and index types**.(支持更多的数据类型和支持自定义类型)

## Drawbacks 

 - Postgres is still less popular than MySQL(为什么?)
 - Postgres forks a new process for each new client connection which allocates a non-trivial amount of memory (about 10 MB).(MySQL好像是单线程的)
 - Postgres is built with extensibility, standards compliance, scalability, and data integrity in mind - sometimes at the expense of speed. Therefore, for simple, read-heavy workflows, Postgres might be a worse choice than MySQL.(高级特性是有代价的)

## Summary

  简单了解后：性能上差不多,不需要根据性能的不同选择Postgres。更多的是因为Postgres拥有更多的拓展性。还有license。
 
## 参考

 - [mysql-vs-postgres](https://developer.okta.com/blog/2019/07/19/mysql-vs-postgres)
 - [mysql-vs-postgres](https://www.2ndquadrant.com/en/postgresql/postgresql-vs-mysql/)
 - [PostgreSQL中文手册](http://www.postgres.cn/docs/12/)
 - [PostgreSQL 与 MySQL 相比，优势何在？](https://www.zhihu.com/question/20010554)