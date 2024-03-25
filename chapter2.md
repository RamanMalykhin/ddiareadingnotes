- All data systems (all IT systems really) rely on a multi-level structure of abstractions. A non-exhaustive, simplified example:
  - An application that processes data (say, an ETL pipeline) has some classes and functions in its code - these are abstractions that are only meaningful to its own logic, but under the hood these abstractions hide the dababase-specific logic of the database the pipeline ingests the data from and loads to.
  - The databases, which expose abstractions such as tables or documents, under the hood use and rely on abstractions that are exposed by the OS or intrepretators of programming languages they are written in, that handle actually storing and reading bits/bytes in system memory.
  - OS or PL interpretators provide to applications that use them abstraction that deal with storing information, but under the hood, rely on abstractions provided by machine firmware, which hides away from them the details of how bits are represented by pulses of electrical current on hardware.

- Relational databases
  - represent data as a collection of relations (collection of tables, which store data in rowsand columns)
  - Defined by E. Codd in 70s, industry standard in business data processing by 80s, still widely used
  - Quite general, can be used both for transactional processing and analytics.
    - Example from modern AWS: Aurora is MySQL optimized for high-troughput OLTP, Redshift is PostgreSQL optimized for analytics.
  - Very strict schema adherence, less mess but more developer lift to properly handle data flowing in and out 
  - ORM - object-relational mapping
    - Code that maps from RDBMS abstractions (tables) to business-logic abstractions 
    - very typical in web dev applications running on top of relational, some frameworks handle it out of the box
  - tend to use IDs as keys and not human-readable information
    - no need to update if human-readable info changes
    - disambiguation (e.g. possible to have multiple cities with exactly same name)
  - if data involves lots of many-to-many relations, much better than documentary (but graph is ideal)
  

- Document-based databases
  - sometimes referred to as NoSQL, but the concept is much older
  - non-relational, only 1 table, not ideal in case of many-to-many relations
  - documents are typically json-like
    - tree structure, only 1 query needed to retrieve all levels of depth 
    - many-to-many relations cause duplication
  - easier to scale horizontally
  - no schema enforcement (schema inferred on read instead of enforced on write), hence less rigid

- Querying and language
  - Standard for relational is SQL
  - Declarative (how the result must look), not imperative (what to do)
    - Neccessarily higher level, lots of assumptions on the part of the query optimizer
  - Mapreduce
    - Paradigm, not language
    - very widespread in big data, only usable with distributed processing, with master node and multiple worker nodes
    - Composed of map procedure by master (filter and splitd the data) and reduce procedure by workers (summarize on splitted partitions)
      - More accurate: map (filter) by master, shuffle (redistribute data based on map) by workers, reduce (summarize based on distributions) by workers
      - inspired by functional programming, both map and reduce are stateless and have no side effects, can re-run if failed
    - Advanced implementations like Spark can expose a SQL-like DSL but run mapreduce under the hood

- Graph databases
  - much better at representing data with many-many connections than relational or document-based
  - instead of storing different tables and inferring the relationships by foreign keys, graph dbs store relationships directly
  - some examples:
    - neo4j
    - Amazon Neptune
    - 