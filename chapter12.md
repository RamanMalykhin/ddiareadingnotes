This chapter, unlike previous ones, contains not the description of the state-of-the-art, but the author's (Martin Kleppmann's) opinions about best practices and how things should be.

There is a very wide variety of things that can be done with data, and a variety of stakeholder requests. It is generally not a good idea to look for a silver-bullet tool that will include all the needed functionality out of the gate - it is usually more likely to get good results by combining tools that do one job well. This is of course at the key of OLTP-OLAP distinction, but goes beyond too - for example, a general-purpose OLAP database may have some support for fulltext search indexes, but if this is an important functionality, it may be a good idea to implement a separate search system. Data integration between these systems is achieved via ETL, which is where stream and/or batch processing is used. 

One special challenge that appears as systems grow is causality of events. When the system is small, its needs can be served by one transactional database with a single leader. But when they grow, logs get paritioned, applications get distributed geographically with different leaders, and systems start getting split into microservices which lowers the interoperability. There are no, as of writing the book, good answers to preserving causality between such events, but logical timestamps or logging the state of the system at the time of the transaction, may help at the cost of added complexity.


Both batch and stream processing are useful to maintain disparate systems connected and integrated. When designing these pipelines, it is extremely useful to build them in a way to make it possible to "replay" older data - for backfilling, but also so that complex evolution of schemas and creation of new derived data is possible. 

An idea called "lambda architecture" exists where for a given downstream data source, streaming is used to update it approximately immediately, and batch is used to update it precisely with some delay. This comes at a cost of significant implementation effort and compute expense. However, some inspiration from this idea has made it into more mainstream approaches, and resulted in support for replaying and for querying HDFS added to some message brokers/stream processors, the importance of guaranteeing exactly-once processing, and for windowing on event time and not processing time (which is meaningless when streaming historical data and aggregating on a window).

Philosophically, the totality of all data driven applications within an organizations (transactional DBs, analytical DBs, search indexes, dataflows that link them) can be viewed as one big database, with different indexes, triggers and storage procedures. The author believes that what could enable significant improvement in designing such systems is a universal high-level language for declaratively composing whole systems. As of writing the book, such a thing does not exist.

Another potential source of inspiration for large-scale data systems is spreadsheets, where it has long (since VisiCalc was introduced in 1979) is being able to reliably express derived data as formulae. This is somewhat doable within databases as stored procedures and views, and across (SQL-compatible) databases with DBT, but still not universal. The author believes what would bring such a vision closer to reality is the paradigm of always keeping all relevant data inputs within reach of compute that produces a derived output (e.g. if a billing service needs to compute the bill in a currency different from the one on the invoice, it must have relevant exchange rates locally), and keeping it always updated by subscribing to update messages from upstream data providers.

Data within an organization follows two processes, which together compose the totality of the path that data undergoes from one end-user to the other - a "write path", which is all of the processes that are performed based on the new record appearing (perhaps as a transaction in the main OLTP store), and a "read path" which is all the processes that are performed when the end-user on the other end makes a request to read data that is downstream of that record. Role of various caches, indexes, materialized views and other tools is shifting the boundary between these paths. Shifting them in various ways presents different options for improving system performance.
For example, typical web development pattern is keeping data on the server, and keeping clients (e.g. browsers) thin and stateless, forcing them to rely on the server to fetch data. This makes client network-bound in terms of performance, and useless without a network connection. In terms of "paths", this means that the write path never extends beyond the compute that the organization controls, data only goes downstream to clients as part of the read path. Extending the write path all the way to client devices and making them stateful would solve these problems. However, this would require fundamentally changing from a model based around HTTP request-response pattern, and towards a message-driven dataflow extending all the way through the org, which imposes steep requirements for performance of all analytical applications that produce derived state. 

# Aiming for correctness 
    # The end-to-end argument for DBs
    
    # Enforcing constraints 
    
    # Timeliness and integrity 
    
    # Trust but verify


# Doing the right thing 