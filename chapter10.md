Processes that automatically transform data are sometimes referred to as dataflows. Doing this on schedule as opposed to immediately when a new record comes in is referred to as batch processing. 
These processes can be understood as inheriting the design philosophy of Unix tooling: the ability to pipe outputs of one command to the other, and solving complex problems in this way, by flexibly chaining different commands. In the world of big data, a popular approach to solving these challenges is MapReduce. 

MapReduce approach is based on splitting a very large amount of data into parts and processing these parts in parallel with a number of independent processes. The order of execution roughly is:
    - Read records
    - Extract needed data from records, by key that will be used for partitioning (this is the "map" stage).
    - Sort output (this is implicit)
    - Perform the computation on the partitioned blocks of data. (this is the "reduce" stage)  

MapReduce was formulated (as an idea and definition) at Google, and then its implementation became an ASF (Apache MapReduce) project. It was folded into the Hadoop ecosystem and used HDFS (Hadoop Distributed File System) for inputs, outputs and checkpoints. Original implementation required users to provide their own Map and Reduce functions, and had no built-in support for doing things that required more than one round of map-reduce operations (e.g. count occurences of key, then produce the most frequent one). Scheduling one after the other was the original usecase for workflow dependency schedulers like Apache Airflow.  
Higher-level frameworks on top of that (Hive, Pig, Spark, etc) appeared later, and introduced the ability to perform more complex operations, requiring more than one map-reduce round. 

There are a few key challenges that batch processing framework implementers (and users) deal with:
    - Partitioning: how to split the data into blocks so as to pass to mappers. 
    - Fault tolerance: the amount of data processed in the batch way is typically large, and failing and restarting a job is an expensive and disruptive event. The standard approach to tolerating faults in batch processes (used by e.g. Hadoop) is checkpointing: writing intermediary transform stages to disk. This, however, io-bounds the performance. More modern implementations (e.g. Spark) trace transformation graphs to re-run only the failing part of the job.
   
Another challenging part is performing joins. Joins become less trivial when datasets are distributed in chunks across multiple machines. There are three approaches, with different implications for performance and applicability.
    - Sort-merge join is when all records that belong to a certain key, from both joined tables, are transferred to a certain executor and handled by it. This causes an exchange of data among the executor processes, and is usually a slow (but sometimes unavoidable) operation. Spark refers to this exchange as a shuffle.
    - Broadcast hash join is when one of the two joinable tables is small and doesn't need partitioning. It is then entirely loaded into a hash table and provided to all executors, who use it for joining with the parts of the larger table they already have.
    - Partitioned hash join is when two inputs just happen to already be partitioned the same way with all data on the same executor. This allows for the expensive exchange/shuffle to be avoided.  