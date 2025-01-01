This chapter, unlike previous ones, contains not the description of the state-of-the-art, but the author's (Martin Kleppmann's) opinions about best practices and how things should be.

# Data integration
There is a very wide variety of things that can be done with data, and a variety of stakeholder requests. It is generally not a good idea to look for a silver-bullet tool that will include all the needed functionality out of the gate - it is usually more likely to get good results by combining tools that do one job well. This is of course at the key of OLTP-OLAP distinction, but goes beyond too - for example, a general-purpose OLAP database may have some support for fulltext search indexes, but if this is an important functionality, it may be a good idea to implement a separate search system. Data integration between these systems is achieved via ETL, which is where stream and/or batch processing is used. 

    # Batch and stream processing

    # Unbundling DBs

    # Designing around dataflow

    # Observing derived state 

# Aiming for correctness 
    # The end-to-end argument for DBs
    
    # Enforcing constraints 
    
    # Timeliness and integrity 
    
    # Trust but verify


# Doing the right thing 