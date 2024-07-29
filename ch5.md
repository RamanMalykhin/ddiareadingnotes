Replication is keeping the data copied across different instances of the DB. Reasons:
- Geographic distribution (like a CDN)
- Resilience to node failure
- Horizontal scaling for increased throughput (usually read)

The single biggest challenge to replication is managing the balance between data quality, aka consistency (all nodes have really the same data) vs availability (even if some data is bad, the system can operate and accept queries). This balance comes up in almost every choice made by the DB designer.

One option is leader-based aka master-slave replication. One node is leader and accepts all queries and writes the write queries to local storage. It then sends the copies of what it wrote to the followers.
- Replication can be synchronous or asynchronous:
	- Synchronous: the query is not considered complete until the change is communicated to some, or even all, follower nodes. This ensures better data quality, but is slower.
	- Asynchronous: the query is only written to the leader and is considered complete, then it propagates to followers in the background. Faster, but weaker durability (may lower data quality if leader goes down before its propagated the changes.)
- Node fail handling:
	- Followers catch up using their log of transactions and log of transactions on leader
	- Leaders must be elected from the remaining nodes, and the system must be reconfigured to route traffic to new leader. Many issues may arise
		- Data loss from failed initial leader
		- Failure to sync autonicrementing keys: may cause data mismatch on join, which can result in e.g. private data leaks
		- Split-brain when multiple nodes believe they are leaders
- Log implementation
	- SQL statements: simple but problematic with nondeterministic operations
	- WAL: byte-level, quite resilient and popular, but tied closely to a specific db engine
	- Logical row-based: kind of between prev 2, records operations on specific rows while retaining some cross-compatibility between db engines
	- Trigger-based: external logging application, e.g. Oracle GoldenGate
- Leader-based dbs can read-scale, meaning, writes are handled by the leader, but reads are handled by replica nodes. This makes availability a lot better, but replication lag becomes a big issue if replication is async (recent writes not yet propagated to nodes from which there are writes.)
	- One option is read-your-own-writes. When a client has written something, they read from the master, or they may keep the timestamp of their last write and their read is redirected to a replica that is known to be in-sync as of that timestamp.
	- Users who never write are also problematic: they may see a value (e.g. comment) when reading from one node, but not the other (where the write has not yet synced). Preventing this is called "monotonic reads", and is usually guaranteed by assigning the user to a given replica that they always read from.
	- A bigger problem is temporal dependency between reads (when some write must have happened before some other write, e.g. comment and reply.) Guaranteeing that this problem is solved is called consistent prefix reads. It is achieved via special implementations that keep track of causal relations.
	
	
Sometimes when leader-based approach seems too limited, multi-leader systems are adopted (they still have read-only followers, but >1 node accepts writes).
- Usecases:
	- multi-datacenter deployment for better resilience and (if traffic geographically distributed) performance
	- Clients that operate offline, e.g. calendars (each client is a kind of leader)
- Single biggest problem: write conflicts. When concurrent writes can happen against a single record, it is hard to know which write to keep final. Approaches:
		- Conflict avoidance (by assigning writes to specific leaders, based on what depends on business logic)
		- Various automated conflict-resolution systems, value merging, etc.
			- Last Write Wins relies on timestamps or clocks, but timestamps are unreliable
			- Special data structures to detect and resolve causal relationships
		- Custom conflict-resolution applets
- Options for replication topology:
 	- Circular: one node passes to next. Fast but if one node fails the circle is broken
	- Star: all nodes talk to one central one (root). Worse speed, a little better fault tolerance
	- All-to-all (self-explanatory): great resilience, but replication lag becomes a serious problem.

Sometimes yet, a leaderless system is used: all replicas accept reads and writes. To guarantee resilience, writes and reads are sent to multiple nodes. The number of nodes that must respond successfully to a transaction (be it read or write) is called a quorum. Value version systems are used to determine which value is more recent.
- Handling restoration after node outages is trickier than with leader-based systems. Two approaches exist:
	- Read-repair: client remembers what version he wrote last, upon read the nodes that returned outdated values are instructed to update.
	- Anti-enthropy process: background job exists that checks all values across nodes and forces updates for outdated ones.
- Quorums follow a formula: `w+r>n`, where `w` is the number of nodes that are queried with write requests, `r` is the number of nodes that are queried with read requests, and `n` is the number of nodes that are storing a given value (this is also all of the nodes of the cluster, but only if the database is not partitioned.) When this holds true, there is necessarily an "overlap" in the number of nodes that are written to and read to, meaning that a client is guaranteed to receive at least one response with the most recent value - others may never come (node failure) or be outdated. This still has some problems:
	- If sloppy quorum was used to increase availability (writes were redirected to non-n partitions because n partitions were unavailable), the guarantee of most recent data is broken.
	- Concurrent writes still cause conflicts.  