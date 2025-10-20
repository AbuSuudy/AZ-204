# Cosmos
Cosmos DB - Under the hood https://learn.microsoft.com/en-us/azure/cosmos-db/global-dist-under-the-hood

Cosmos is multi model No SQL (not just SQL) that designed to globally distributed to multiple nodes. 

Azure Cosmos DB offers multiple database APIs, which include NoSQL, MongoDB ( document structure), PostgreSQL (Relational) , Cassandra (column-oriented schema), Gremlin (graph), and Table (key values). API for NoSQL is native to Azure Cosmos DB so get the latest and greatest sooner compared to the other API's

The advantages here: 
- Low latency: Data can be close to where it's needed e.g. the API or user 
- Highly available due to having multiple copies 
- The way partition keys are used it could scale horizontally allows for high scalability because load is spread across multiple nodes. Instead of having one large SQL relational database. 

> [!NOTE] 
> You're able to shared a relational database across multiple servers to allows for horizontal scaling, but you will need coordination with syncing schema changes and directing data to specific instances and having an application that would distribute the shared based on *shared key.* Cosmos allows managed environment that handles this for you. Also schema not being as strict compared to typical relational databases.
## Cosmos Architecture 
**Azure Database Account** -  account contains all of your Azure Cosmos DB resources: databases, containers, and items. Database account can have multiple databases.

**Database** - is similar to a namespace. A database is simply a group of container.

**Container** - is where data is actually held. Data is stored on one or more servers called _partitions_. Which use *Partition key* which help Azure Cosmos DB distribute the data efficiently across partitions. You can also use the partition key in the `WHERE` clause in queries for efficient data retrieval. When an app using Azure Cosmos DB elastically scales throughput on an Azure Cosmos DB container or consumes more storage, Azure Cosmos DB transparently handles partition management operations (split, clone, delete) across all the regions.

![](Images/hierarchy.png)

> [!NOTE] 
> To scale cosmos db more partitions are added which is managed by cosmos.

- **Logical Partition** -  A logical partition is a set of items that share the same partition key e.g `"city" : "London"` . This data could be stored in multiple physical partitions.

![](Images/logical-partitions.png)

**Physical Partitions** - are an internal implementation of how Azure cosmos store your data. A physical partition is implemented by a group of replicas, called a _replica-set_.

![](Images/Pasted%20image%2020251020201015.png)

Each replica set has 4 nodes which all have the copy of the data:
1) **Leader** - process the write operation
2) **Followers** - Gets updates from the leader
3) **Forwarder** - Is also a follower, but also has the responsibility to forward the updated data to the leader of the replica set in to other regions.

Due to replica set we have 4 copies our data our local region. If we want have data to be closer to our users we deploy a replica set of 4 nodes to that region. Cosmos managed environment with the forward will allow data to be kept up to date depending on the consistency model you choose.
## RU (Request Unit)
Request unit represent: CPU, Input/Output (IOPS) and memory required to preform a database operation. Measures cost based on throughput (Request Units per second, RU/s). Reading single item using partition key and id should be 1RU if it's under 1kb in size. 

A 1KB text example is approximately the length of a short email, a few paragraphs, or around 160-200 words in plain English. 

Types of database determines what RU are charged: 
- **Provisioned throughput** : You manage RTU for your application in multiple of 100 RU per seconds. 
- **Serverless** : Billed on RU you use 
- **Autoscale**: Scale RU based on usage

![](Images/request-units.png)

## Consistency Level
Consistency model determines how data is replicated across nodes.

**Strong Consistency** - Data is available in all nodes in the same time. Strong consistency increases write latencies because data must replicate and commit across large distance and also reduces availability during failures because data can't replicate and commit in every region.

Latency is for one request, multiple request could be made which increase overall time exponentially. 

![](Images/Pasted%20image%2020251020164502.png)

**Weaker Consistency**- They will eventually be in-sync, but will take a longer time. Offers higher availability and better performance, but it's more difficult to program applications because data might not be consistent across all regions.

Based on your applications you will make different trade offs on what model to choose.

![](Images/five-consistency-levels.png)

**Strong Consistency** - reads are guaranteed to get the most recent commit, but since data will need to commit to all regions so could have some latency issues based on distance to regions. 

![](Images/Pasted%20image%2020251020153855.png)

**Bounded staleness consistency (Read/Write Replica)** - Similar to strong, but you could configure how much the read copies could lag behind:
- **K** - number of updates it could lag behind the writes
- **T** - Time interval that it could lag behind the writes

Bounded staleness is primarily beneficial to single-region write accounts with two or more regions. Once you're getting close to the time window Cosmos will throttle and not accept any writes. To allow replication engine to catch up.

![](Images/Pasted%20image%2020251020154033.png)

**Session Consistency** - The default for cosmos db. It ensures that **within a client session**, a user will always read their own writes. When a client connects to cosmos is session token is issued. This token passed onto Cosmos to ensure the what you've written is returned.  This more of client side model.

![](Images/Pasted%20image%2020251020162518.png)

**Consistent Prefix** -  Reads will lag behind your writes. So you won't see your changes straight away like session consistency, but the order will still be maintained.

![](Images/Pasted%20image%2020251020162611.png)

**Eventual Consistency** - Doesn't guarantee ordering, but data will eventually all be there.

![](Images/Pasted%20image%2020251020162749.png)

> [!NOTE]
> If your Azure Cosmos DB account is distributed across _N_ Azure regions, there will be at least _N_ x 4 copies of all your data. You have 4 replicas to ensure high availability, fault tolerance, and strong consistency guarantees. Any times you make changes your data is committed to 3/4 replicas. The last one get hydrated and gets updated in the background.
> 
> **Strong and Bounded** : Reads from two replicas. When you read both replicas you can read logical sequence number (LSN) and compare them. If they're the same value you're reading the strongest set of data that has been written in. If they're not the same you return the data with the highest LSN (the most recent data). Since it reads from two replicas the RU cost is doubled.
> 
> **Session, consistent prefix, eventually**: Read only from a single replica
> 



