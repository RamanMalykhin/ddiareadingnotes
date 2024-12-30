Batch processing systems have many benefits, but a significant issue: they, as the name implies, artificially split data into batches (usually by time). But in reality, transaction data doesn't come in a big batch daily or hourly: it comes every second, and many systems require derived data from these transactions to also be updated continuously. Processing data reactively as it comes, instead of in batches, is referred to as streaming. 

In streaming, in place of a filesystem or distributed file system, transactions need to come in the form of messages. Different approaches to sending them exist.
    - There are messaging protocols that can send them from the transaction data source, and especially when used with multiple downstream receivers, this is referred to as "publish/subscribe" or "pub/sub" model. Pub/sub systems tend to differ with respect to two points:
        - What happens when consumers are slower to process than producers are to send. Messages can be dropped, buffered in a queue, or the publisher can be backpressured (prevented from sending more). 
        - Whether crash of any parts of the system causes message loss. It can be acceptable in some systems, and really depends on the real-world usecase. 
    - There are cases where a whole message sending system or application is used to distribute messages and track them. They are referred to as message brokers. The most popular one is Apache Kafka. This is usually used when the messages are asynchronously processed and that is part of the business logic: some of the message receivers may use the message immediately, some may process it later. Brokers usually manage message loss well by implementing so-called DLQ (Dead Letter Queue) which tracks messages that failed to be delivered. They come with some performance penalties compared to more lightweight approaches. Brokers can deliver messages to all of the consumers ("fan-out"), or deliver them to one and let consumers manage the load by e.g. internally splitting across multiple machines. Brokers usually require "acknowledgements" from clients that a message has been processed successfully.
    - In case where message loss is acceptable when traded for low latency, direct messaging from producers to consumers over e.g. UDP or API endpoints (an API endpoint that receives requests automatically as messages come into its user's system is referred to as a webhook) is acceptable. 

Streaming systems are frequently used to consume transactional data from a database, and different approaches to it exist. The most obvious one is reading directly from the database log, but database logs are and historically were designed as internal system components, to be used for debugging purposes, but not as a frequently-used interface to the system. Systems to capture data from databases are referred to as CDC (change data capture). They can work with a variety of approaches - some use database triggers, some use replication logs. CDC systems are usually designed for a particular database and implement capture logic specific to that database's internals. 

#TODO
Use of time 

Joins

Fault tolerance and exactly once semantics (microbatch, checkpointing, txn, idempotence)
