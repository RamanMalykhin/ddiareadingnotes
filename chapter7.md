A transaction is an old concept related to DBs, mostly applied to Relational DBs. A transaction is a few operations combined into a single unit, which comes with a few guaranteees that individual operations would not. These guarantees are known as ACID. The DB that implements transactions with these guarantees is called ACID-compliant. 
- ACID stands for:
	- Atomicity. This means that the transaction can either succeed entirely (commit), or fail entirely (abort). It cannot be applied partway.
	- Consistency. This is not the same consistency as in CAP theorem, or in replication. Consistency in ACID means that the transaction must not be able to push the database into a "bad" state. The definition of a "bad" state is unfortunately vague. It may mean just not being able to invalidate the schema. It may mean not being able to invalidate the application's business logic.
	- Isolation. Each transaction must be able to run as if it is the only transaction running, other transactions happening concurrently do not matter and should not be able to intermingle with each other. There are some isolation levels, some of which provide weaker and some stronger guarantees.
	- Durability. After a trasnaction has been committed, is result must be available in non-volatile memory, and must be available after a system fails and must recover. In replicated systems, it usually means that a trasaction is successful after its result has been replicated across some replicas. The concept is to be understood as relative: no system is perfectly durable. E.g. if Planet Earth were to be destroyed, most backups of most systems would be unavailable, together with the systems themselves.

Isolation and atomicity in RDBMS are usually on TCP-connection level: everything between `BEGIN TRANSACTION` and `COMMIT` statements is understood to belong to the same transaction. This doesn't always work with non-relational databases.

Some databases can provide lighter guarantees for single-object operations, e.g. compare-and-set (only change the value if it has not been concurrently changed by something else). These are sometimes called "lightweight transactions" for marketing purposes, but the terminology may be confusing.
However, the key philosophy of transactions is about providing guarantee for multi-object operations as well. This is something not all non-relational database do, and not all distributed relational databases do either. Leaderless-replicated systems tend to guarantee the least.

Particular problems with transactions arise because of concurrency issues (race condition). They are hard to catch because any single given operation affeecting any particular record is acting perfectly normally: the problem arise when the timing is unlucky and they coincide, causing non-deterministic results for the affected data. 
Some examples:
- Dirty read: 
  - reading writes of another transaction before they have been commited
- Dirty write: 
  - overwriting writes of another transaction before they have been committed 
- Read skew:
  - Not seeing the full picture between transactions (e.g. subtraction from one account but no addition to the other)
- Lost update:
  -  Two clients are reading, modifying and writing the data at the same time. One reads before the other but commits after, so the data is lost between updates.
- Write skew:
    - Transaction reads something, makes the decision based on the content, and writes the outcome of the decision: but by the time it wrote the premise is no longer true.
  - Phantom read
    - Happens when search (e.g. `WHERE`) selects based on some value, but the concurrent update removes it from some of the results, making the search no longer valid. 
The most desirable solution to race conditions are "serializable" transactions. This does not have anything to do with JSON or XML serializability - rather, that means that the application can pretend that transactions are not concurrent and are running one after the other.
Unfortunately, true serializability is performance-expensive and some databases provide weaker guarantees about isolation. Some options for weak guarantees:
- Read-committed transactions
  - Very popular isolation level. Standard in Oracle, Postgres, and others. These only prevent dirty reads and dirty writes. 
  - Different ways of implementation exist. Most straightforward is row locking, but it can harm performance if one long-running read transaction locks up a wide range of rows. More modern way is snapshotting
- Snapshot isolation
  - Prevents dirty writes, dirty reads, read skew, sometimes lost updates, and some straightforward kinds of phantom reads.
  - A more generalized version of snapshot-based prevention of dirty reads: multiple versions of an object are kept in memory. This is called MVCC (multi-version concurrency control).
    - The snapshots presented to the transaction is defined by visibility rules. Typically only writes by transactions that have started AND committed before the start of a given transaction make it into the snapshot that is visible to this transaction.
    - There are some complications related to indexing in databases with MVCC.
  - Lost updates are usually prevented using forced atomicity for write operations (when the transaction includes a read-update-write cycle, the lock on the values is hard). But sometimes an explicit lock is requred by the application (`SELECT FOR UPDATE`)
  - Lost update detection is especially hard in distributed databases where different version may exist on different nodes. Sadly, Last Write Wins is a widespread option for  conflict resolution and tends to result in lost updates.
- Serializable tranasctions
  - Note that this relates to single-machine, non-distributed databases, and distributed are covered in a different chapter in the future.
  - Strongest isolation level. Protects against all race conditions, with a serious performance cost.
  - Most straightforward way to implement this is actual serial execution by forcing all concurrent operation into a single processor thread. This was infeasible for a very long time, but cheaper RAM allowed to keep the dataset in memory which makes repeating a lot of transaction cheap enough. This also can make use of stored procedures, which are pre-defined queries or scripts submitted to the database ahead of time. This really only works well when the transactions are small, the data fits in memory, and the throughtput is low enough to work on a single core.
  - Oldest way to implement serializability is 2PL (two phase locking). The principle is that once some transaction is writing to some object it has exclusive access, and no other transaction is allowed to write OR read it. 
    - This causes deadlocks - situation where two transactions are watiting for each other to release locks on different data. The query engine detects such cases and forces one of the transactions to cancel and retry.
    - Performance of 2PL is rather bad due to a high number of locks and frequent deadlocks. So many tweaks exist to optimize locking: predicate locks (based on a query) and index-range locks (simplified approximation of predicate locks that is much better for performance of lock checking, but can lock up larger volumes of data than necessary).
  - A more modern way of implementing serializability is SSI (Serializable Snapshot Isolation). It is rather advanced. It uses MVCC transactions to proceed without any locks but checks them at commit, and aborts them if the transaction was not serializable because of outdated MVCC snapshot usage
    - The rate of aborts is quite high if the database is really very highly concurrently loaded with high contention, but practice shows it can be a viable option in many cases.
- It is important to note that some databases can use multiple isolation levels for different queries. E.g. long OLAP queries that are read-only can run on snapshot-level while write transactions require serializability.

