Databases and their schemas evolve, and so must schemas. Such changes also impact other data-driven systems (message passing systems, APIs, etc.)

Databases:
- Relational dbs are hevily schema-driven and require an opinionated and clear schema to start with. Only 1 schema is in force at any given point.
- Non-relational dbs are much less opinionated about data, and in practice enforce only schema-on-read. So at any given point a document db (for example) can contain data to which multiple schemas apply. 

Applications:
- Code also must change as data/schemas evolve. This is typically challenging but in different ways.
    - Client-side apps are at the mercy of the user
    - Server-side apps are in constant use and must either go down for maintenance, or have a rolling update strategy

Compatibility must be maintained both ways:
- Forward compatibility
    - Data written by new code can be read by old code
- Backward compatibility 
    - Data written by old code can be read by new code

All these issues are at the core and have an outsized impact on how data is turned into bits - in other words, encoding / serialization.

Few serialization approaches:
- Language-specific (`java.io.Serializable` `pickle` in Python, and so on) 
    - Not cross-portable between languages
    - Insecure (may require insantiate arbitrary classes )    
    - Typically not very efficient, sometimes very inefficient
    - Usually no support for schema versioning

- Popular string-based formats (JSON, XML, CSV)
    - Human-readable at the cost of size
    - Float encoding is complex and has pitfalls
    - Schema support doesn't exist for CSV per se, exists for JSON and XML, but schema languages are complex and hard. CSV formal escape rules are not always well implemented in all parsers.
    - Binary variants exist (MessagePack, BSON, BJSON, WBXML, ...)
        - Space savings are not great
        - No schema prescription
        - Do have uses, but niche

- Thrift/Protobuf
    - Popular with RPC APIs
    - Binary encoding, relies on code generation to read/write and enforce data types on client and server
    - Much more space savings compared to binary variants of JSON/XML
    - Schema evolution supported, relies on editing definition files (e.g. `.proto`). Only compatibility issues - adding a new field as required (reading old data will break as the field will be gone)

- Avro
    - Row oriented data storage format, exists for storage and APIs (Avro-RPC)
    - Separates writer schemas and reader schemas, embeds writer schemas in `.avro` files - reading without knowing the schema is effortless.
    - code generation is optional and only needed for statically typed languages
    - very compact


Dataflow
Now we can discuss applications of serialization in various cases of passing data between systems (of course, it will be serialized in this case).
- Dataflow through databases
    - Most DBs use a specific serialization protocol that can take care of this  
    - Application-level support and tweaks are rarely needed
    - Important to keep in mind: when doing data dumps, it's best to not persist historical schemas, and just write it with the most recent one, assuming the handling is good (there is no mistyped data)
- Dataflow through service calls
    - When calls are via HTTP, it's usually called a web service. E.g:
        - Browsers
        - Middleware (microservices)
        - Commercial inter-organization APIs
    - Two popular approaches exist and dominate the sphere:
        - REST - closely coupled with HTTP as a protocol, uses it for auth and cache control, does not try to hide the fact that the call is remote. JSON-payloads are not opinionated about schema.
        - SOAP - over HTTP but does not rely on it, XML-driven. Uses own WSPL language for payload schemas.
    - All this is based on an old idea of RPC - Remote Procedure Call, aka calling a process on another machine just as if it were on the same one. This has serious issues.
        - Network problems happen and introduce timeouts and connection breakages
        - Memory sharing is obviously not possible and needs serialization
        - Idempotence (statelessness) is hard because of potential connection breaks and duplications.
    - Some new formats exist
        - gRPC over Protobuf
        - Finagle over Thrift
        - Avro-RPC over Avro

Some other dataflow cases
- Actor model
- Message brokers