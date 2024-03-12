- Modern applications are frequently data-intensive, not compute-intensive
  - More emphasis on capacity to process complex data, reliability, backfill capacity, etc, than raw CPU output

- "Data systems" is a fuzzy abstraction
  - In reality caches (e.g. Redis), message brokers (e.g. Kafka), relational DBs (e.g. MySQL), non-relational DBs (e.g. Mongo), search engines (e.g. ElasticSearch) are meaningfully different systems
  - But the distinctions are messy and it is possible and sometimes feasible to use a given system in a different way than what it was designed to do (e.g. using ELK stack as an analytics DB/BI platform), so it makes sense to think about "data systems" broadly.


- A data-intensive system should satsify requirements from many different domains to be successful
- 3 are key:
  - Reliability
    (how well does a system tolerate faults, errors and other unexpected situations)
    - Hardware faults
        (only partially manageable, key is redundancy/secondary machines/replication)
    - Software faults
        (hard to manage, but good system design, a mature software development process, and observability all help)
    - Human errors
        (keys to managing are extensive testing, decoupling dev environments from prod, monitoring/observability and clear recovery procedures)
  - Scalability
    (how well can a system adapt to ever-increasing load)
    - Key to describe performance properly
      - Throughput (how long it takes to process N records)
      - Response time (how long a request waits for response from API)
      - Usually measured in percentiles - e.g. how long the longest request in 10000 (99.99th perc) takes
    - Scaling can be vertical (moving to a more powerful single machine) vs horizontal (adding more nodes to a distributed system)
      - Moving from one type of architecture to another is usually a big hurdle and takes a lot of dev time
    - Some systems are "elastic" which means they are able to auto-scale, usually horizontally (automatic spinup of more machines). But sometimes this is harder to maintain than manually scaled systems.
  - Maintainability
    (how easy is it for humans to work with the system)
    - Operability
        (how easy is it for OPS teams to perform BAU tasks)
    - Simplicity
        (how easy is the design of the system and its architectural components to understand)
    - Evolvability
        (how easy is it to introduce changes to the system - legacy is a big problem here)

