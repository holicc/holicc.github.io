## What is GFS?

Basiclly gfs is a storage application,deployed within Google as the storage platform.

Google file system a **scalable** **distributed** file system for large distributed data-intensive applications. It provides **fault tolerance** while running on inexpensive commodity hardware, and it delivers high aggregate performance to a large number of clients.

## Using for ?

Google File Sys- tem (GFS) to meet the rapidly growing demands of Google’s data processing needs.

## Problems ?

**First**,when hundreds or event thousands client or server application are runing , they may be crashed at anytime by any reasons.So the gfs must be automatic recovery from crash.

**Second**,with the data growing ,it is hard to manage the data sets.As a results ,we must guarantee the performence such as I/O operation and block sizes have to be revisited.

**Third**,consider reading and writing,using data.For example the mutable data by appending new data rather than overwriting existing data,share data characteristics. Some may be archival data. Some may be in- termediate results produced on one machine and processed on another, whether simultaneously or later in time.

**Fourth**,elegent API design make system flexible and extendable.Also provided atomic append API to keep system stable and safty.

## Architecture

GFS has three important role:*Single Master* *ChunkSerfers* *Clients* 
![Figure 1](/images/figure.png)

Single Master: Infomartion center hub.
> The master maintains all file system metadata.
> The master peri- odically communicates with each chunkserver in HeartBeat messages to give it instructions and collect its state

deal with sophisticated chunk placement and replication logic.But master doesn't read or write data to client.

Metadata.

Chunk Locations.

**Operation Log**


chunkservers: just like databases.
> Chunkservers store chunks on local disks as Linux files and read or write chunk data specified by a chunk handle and byte range
> By default, we store three replicas, though users can designate different replication levels for different regions of the file namespace.

client: bridge of master and chunkservers
> Clients interact with the master for metadata opera- tions, but all data-bearing communication goes directly to the chunkservers.


**Neither the client nor the chunkserver caches file data.**

## Consistency Model 

The state of file region:**consistent**,**defined**,the two state combain with two types(serial,concurrent)

just like database's transcations ? 