# ElasticSearch-Note (es version 7.3)

## Intdroduction

  - distributed document search system build in java
  - provides **near real-time** search and analytics for all types of data
  - distributed document store(complex data structures that have been serialized as JSON documents)

### Document

Instead of storing information as rows of columnar data, Elasticsearch stores complex data structures that have been serialized as JSON documents.

Elasticsearch automatically detects and adds new fields to the index. This default behavior makes it easy to index and explore your data—​just start indexing documents and Elasticsearch will detect and map booleans, floating point and integer values, dates, and strings to the appropriate Elasticsearch datatypes.

### Index

Diffrent field type has diffrent index.For example, text fields are stored in inverted indices, and numeric and geo fields are stored in BKD trees. The ability to use the per-field data structures to assemble and return search results is what makes Elasticsearch so fast.

### Shard

There are two types of shards: primaries and replicas.

**Each shard is actually a self-contained index**

By distributing the documents in an index across multiple shards, and distributing those shards across multiple nodes

Elasticsearch automatically migrates shards to rebalance the cluster.

The number of primary shards in an index is fixed at the time that an index is created, but the number of replica shards can be changed at any time, without interrupting indexing or query operations.

The more shards, the more overhead there is simply in maintaining those indices. The larger the shard size, the longer it takes to move shards around when Elasticsearch needs to rebalance a cluster.

## Type(removed sinece 7.0)

In an Elasticsearch index, fields that have the same name in different mapping types are backed by the same Lucene field internally. In other words, using the example above, the user_name field in the user type is stored in exactly the same field as the user_name field in the tweet type, and both user_name fields must have the same mapping (definition) in both types.

This can lead to frustration when, for example, you want deleted to be a date field in one type and a boolean field in another type in the same index.

On top of that, storing different entities that have few or no fields in common in the same index leads to sparse data and interferes with Lucene’s ability to compress documents efficiently.

Alternatives to mapping types:**Index per document type**,**Custom type field**,**Parent/Child without mapping types use join field**

## Summary

-   Near real-time(within in 1 seocond) distruibition search engine.
-   Basic on lucene to build index.
-   Document reprente basic data structed unit,contains kinds of types fields.
-   Index basic search unit contains documents.
-   Shard just like data chunk in distrubit file system.
-   Replica backup of shard spared in nodes.

## Questions

- [x] Why called near real-time ?
- [x] How to make choose shard and replica size ?
- [x] How to optimize query performance ?  
- [ ] How to sperad data in diffrent shards ?
- [ ] The bottom principle of write and read data ?
- [ ] How to communicate between nodes ?