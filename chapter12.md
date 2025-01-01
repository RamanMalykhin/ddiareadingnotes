This chapter, unlike previous ones, contains not the description of the state-of-the-art, but the author's (Martin Kleppmann's) opinions about best practices and how things should be.

# Data integration
There is a very wide variety of things that can be done with data, and a variety of stakeholder requests. It is generally not a good idea to look for a silver-bullet tool that will include all the needed functionality out of the gate - it is usually more likely to get good results by combining tools that do one job well. This is of course at the key of OLTP-OLAP distinction, but goes beyond too - for example, a general-purpose OLAP database may have some support for fulltext search indexes, but if this is an important functionality, it may be a good idea to implement a separate search system. Data integration between these systems is achieved via ETL, which is where stream and/or batch processing is used. 

One special challenge that appears as systems grow is causality of events. When the system is small, its needs can be served by one transactional database with a single leader. But when they grow, logs get paritioned, applications get distributed geographically with different leaders, and systems start getting split into microservices which lowers the interoperability. There are no, as of writing the book, good answers to preserving causality between such events, but logical timestamps or logging the state of the system at the time of the transaction, may help at the cost of added complexity.


Both batch and stream processing are useful to maintain disparate systems connected and integrated. When designing these pipelines, it is extremely useful to build them in a way to make it possible to "replay" older data - for backfilling, but also so that complex evolution of schemas and creation of new derived data is possible. 

An idea called "lambda architecture" exists where for a given downstream data source, streaming is used to update it approximately immediately, and batch is used to update it precisely with some delay. This comes at a cost of significant implementation effort and compute expense. However, some inspiration from this idea has made it into more mainstream approaches, and resulted in support for replaying and for querying HDFS added to some message brokers/stream processors, the importance of guaranteeing exactly-once processing, and for windowing on event time and not processing time (which is meaningless when streaming historical data and aggregating on a window).

Philosophically, the totality of all data driven applications within an organizations (transactional DBs, analytical DBs, search indexes, dataflows that link them) can be viewed as one big database, with different indexes, triggers and storage procedures. The author believes that what could enable significant improvement in designing such systems is a universal high-level language for declaratively composing whole systems. As of writing the book, such a thing does not exist.

    # Designing around dataflow

    # Observing derived state 

# Aiming for correctness 
    # The end-to-end argument for DBs
    
    # Enforcing constraints 
    
    # Timeliness and integrity 
    
    # Trust but verify


# Doing the right thing 