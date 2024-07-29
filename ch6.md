One of the techniques to improve scalability of the database is partitioning: splitting data across different nodes. This means that some of the data is only stored on one machine, and some on the other (different from replication, where THE SAME data is stored on different nodes). 
If the data split across `n` nodes is fair (meaning, data is distributed across partitions in a way that query load is split fairly), this improves db throughput `n` times compared to a single-machine db. Un-fairness of paritioning is called skew, and a parition which skew caused to have unfairly more load than others is called a hotspot.

Several strategies of parititioning data exist. 
- 	Key ranges: Key ranges should be chosen in a way that splits the data evenly: e.g. we cannot keep an english-language encyclopedia split by alphabet, because there are much more words starting with some letters than with others. The DB admin may choose his own partition boundaries, or let the DB do it automatically.
	- Key ranges are pretty effective at range queries as they allow partitions that are out of query to be skipped over.
	- Problem is that it is very hard to avoid hotspots with key ranges in practice. E.g. one may be using timestamps, but if streaming data in real time, most recent partition will always be a hot spot.
- Key hash: Hash functions have a property that they tend to produce random results, which deskews data. Then, ranges are formed based on ranges of hash values (or a few bytes of hash values).
	- This is much more effective at skew avoidance, but loses all the benefits related to range lookup.
	- Compound primary key: compromise between hash and key range approaches. Primary keys are declared based on several columns, one of them is hashed to determine the partition, but indexing is done on the concatenation of all columns of the PK. This still makes range queries on the first column of the PK inefficient, but if the query specifies a fixed value for the first column, range scans are efficient over other columns.
	
	
While hashing keys is effective at hotspot reduction, hotspots still can happen if all the spike in activity concerns one key. E.g. activity spikes in a social network when 1 person with a lot of followers makes a post. 
- Typical solution is splitting the key (e.g. adding 2 random digits) into many to allow them to be partitioned, but this hurts data inregrity (Now, there is a need for additional logic to unify many splitted versions of the key and make consistent dat across them) and requires additional logic.

One more problem with partitioning is when the DB uses a secondary index. There is no guarantee that secondary index will in any way map onto partitions based on the primary index. 
- Scatter-gather approach with local indexes: Each partition will have its own secondary partition index. If the query is performed based on a secondary index value, but does not filter the primary index, values will have to be unified across partitions. This makes the query computationally expensive, and the approach is called scatter/gather. Despite downsides, it is relatively widespread.
- Global index: this option relies on having a secondary index that covers the entire data, and partitioning it, but it can be partitioned completely differently from primary index. That way, a query on the secondary index will not have to query all partitions. However, this makes writes much more complex (in addition to writing the transaction under its primary index's partition, the write operation needs to locate all secondary indexes, in all partitions, and update them too). 

Sometimes, machine failure, data increase and throughput requirement increase cause a need to rebalance partitions across nodes. Typical requirements are: rebalancing should lead the database to a fairly balanced state, rebalancing should not block the db from accepting writes, and the need to move data across nodes must be minimized.
Strategies:
 - Hash mod N, where n is number of nodes. This is a pretty bad strategy as it tends to cause a lot of shifting of keys between nodes.
 - Fixed number of partitions, growing number of nodes. In this case, there are more partitions than nodes. When there is a need to grow the cluster, 1 more node is added and accepts some partitions from existing nodes. This is done until the distribution is fair again. There is never any repartitioning, only movement of partition between nodes. 
 	- Movement is usually done in the background, as movement of partitions is usually long and expensive. 
	- This approach is efficient, but requires a very educated guess from the engineers of the system in choosing a number of partitions. 
- Dynamic partitioning. This is similar to the previous strategy, but also allows to dynamically split off more partitions as the data size grows. The main caveat is starting with a single partition: then, as the dataset grows, it becomes easier to see what columns to partition by.
- Partitioning proportional to nodes. Fix the number of partitions per node, then as new nodes appear, split off some data from existing partitions into it. This usually works with large partition counts. Since data from old partitions is added to new ones randomly, this works for hash-partitioned data. 

Rebalancing can be manual or automatic, but tends to exist on a gradient: fully automatic, system-suggested but admin-confirmed repartitioning, explicit manual configuartion. Fully automatic rebalancing is convenient, but unpredictable and because of that a bit risky.

Request routing is also complex with partitioned systems - it is a subest of a broader "service discovery" problem. In a nutshell the question is - how does a client know which partition can handle its request. Usually separate tools (custom, commercial or open-source) are used.  
- There are several generic approaches to the problem:
	- Allow to connect to any node, if the node happens to be the right one it handles the request, otherwise it forwards the request to the right one.
	- Use a partition aware load balancer, aka routing tier
	- Make clients have awareness of partitioning and require them to make the right requests themselves.
- All approaches rely on a distributed and mutually synchronised understanding of the partitioning. Frequently, a separate metadata-tracking service is used, e.g. Apache Zookeeper. Sometimes dbs come with their own implementation, e.g. MongoDB's config server. Sometimes, the DB may force all nodes to be aware of all partitioning, that is called a "gossip protocol" approach. 