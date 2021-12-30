# Notes on _Designing Data-Intensive Applications_

[Get the book here](https://dataintensive.net/).

- [Part 1: Fundamentals](#part-1--fundamentals)
  - [Chapter 1: reliability, scalability and maintainability](#chapter-1--reliability--scalability-and-maintainability)
  - [Chapter 2: data models and query languages](#chapter-2--data-models-and-query-languages)
  - [Chapter 3: storage engines](#chapter-3--storage-engines)
  - [Chapter 4: serialization and schema evolution](#chapter-4--serialization-and-schema-evolution)
- [Part 2: Distributed Systems](#part-2--distributed-systems)
  - [Chapter 5: replication](#chapter-5--replication)
  - [Chapter 6: partitioning/sharding](#chapter-6--partitioning-sharding)
  - [Chapter 7: transactions](#chapter-7--transactions)
  - [Chapter 8: problems of distributed systems](#chapter-8--problems-of-distributed-systems)
  - [Chapter 9: achieving consistency and consensus in distributed systems](#chapter-9--achieving-consistency-and-consensus-in-distributed-systems)
- [Part 3: Derived Datasets](#part-3--derived-datasets)
  - [Chapter 10: batch processing](#chapter-10--batch-processing)
  - [Chapter 11: stream processing](#chapter-11--stream-processing)
  - [Chapter 12: putting it all together](#chapter-12--putting-it-all-together)

## Part 1: Fundamentals

### Chapter 1: reliability, scalability and maintainability

Just like databases, queues and caches can be thought of as data systems (because the line between
each is blurred in today's ecosystem), we can also think of our own applications as special-purpose
data systems built from generic components that solve more abstract problems.

#### Reliability

A _fault_ is a deviation in the functioning a component from its spec. A _failure_ is a complete
service interruption to the user.

Fault tolerance is best thought of as the design of a system such that the probability of a fault
causing a failure becomes low: building a reliable system from unreliable parts. Designing a system
without faults is not possible.

It's generally preferable to tolerate fault than to try to prevent them, except in the case of
security issues, where the event of a system being compromised cannot be undone.

- Hardware faults: machines and components have a mean time to failure (MTTF). The industry has
  evolved from designing fault tolerance of components (hot-swappable CPUs, generators for backup
  power) to additionally designing systems that tolerate the loss of entire machines through
  software.
- Software faults: they usually cause more problems than hardware faults because they're present
  horizontally across the system. They're harder to predict and usually lie dormant until a certain
  set of circumstances occur simultaneously.
- Human error: systems can be designed, decoupled, monitored and tested so that opportunities for
  human error are as thin as possible. Humans can be trained.

#### Scalability

Scalability describes a system's ability to cope with increased load. It must be described in a
non-binary way. A system may scale up to (for example) 10,000 concurrent users but not 1 million.

Load parameters are used to describe system load. With Twitter as an example, important parameters
are tweets posted per second, home timeline views per second, and the distribution of followers per
user (because each user's home timeline loads the tweets of all its followers).

Armed with the ability to describe load parameters, we can now describe the performance of a system
in its terms: how is performance affected when a specific load parameter increases and the system
resources remain unchanged? How do we need to change system resources to keep performance the same
as a specific load parameter increases?

_Response time_ is the amount of time that passes since a client produces a request and the time
they get a response back. _Latency_ is the amount of time the request is latent, waiting to be
handled.

Response time must be thought of as a distribution and not as a single number, because response time
changes on every request, even if the request remains the same.

[Percentiles](https://en.wikipedia.org/wiki/Percentile) are the most common way to describe a
distribution of response times. The 95th, 99th and 99.9th percentiles are common, also referred to
as p95, p99 and p999.

Percentiles are used in SLOs and SLAs (service-level objectives and service-level agreements)
between customers and service providers to define the expected availability and performance of a
service.

Head-of-line blocking happens when one slow request causes the slow handling of subsequent requests.
This is especially important in services where one user request may trigger multiple calls to
backend services. Tail (i.e. high-percentile) latency amplification is a term used to describe a
slow-down of multiple user requests due to one such backend service call performing poorly.

To cope with increased load, architectures must be reconsidered (usually) on every
order-of-magnitude load increase. One design will not fit all scalability needs.

Designs can _scale up_, or vertically, by moving to a more powerful machine, or _scale out_, or
horizontally, by distributing load across more machines. Single-machine architectures are simpler,
but a hybrid approach is usually good.

_Elastic_ systems add computing resources autonomously and automatically, but other systems must be
scaled manually, and in turn remain simpler.

#### Maintainability

_Operability_ is
vital: [good operations can often work around the limitations of bad (or incomplete) software, but good software cannot run reliably with bad operations](https://blog.empathybox.com/post/19574936361/getting-real-about-distributed-system-reliability)
. To make operations easier, software can provide runtime visibility, support for automation,
documentation, good default behavior and predictability, and it can avoid depending on an individual
machine.

_Simplicity_ must be attempted with the use of abstractions.

_Evolvability_ can be aided by good organizational processes.

### Chapter 2: data models and query languages

This chapters covers general-purpose data models for data storage and querying. Data models are the
most important part of developing software: they represent how we think of the problem we're solving
and stow away the underlying complexity of an application through a clean lens.

#### Relational model versus document model

The relational model
was [proposed by Edgar Codd in 1970](https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf), and
it's the data model on which SQL is based. In it, data is organized in relations (tables in SQL)
where each relation is an unordered collection of tuples (rows in SQL).

Edgar Codd's idea gained traction by the mid-1980s and remains the dominant model today.

Other models have competed with the relational model since the 1970s:

* The network model in the 1970s
* The hierarchical model in the 1980s
* Object databases (where data is represented as objects as used in object-oriented programming)
  since the 1980s
* XML databases (a flavor of the document model) in the 2000s

None of those attempts have lasted. The latest, since the 2010s, is NoSQL.

##### NoSQL

The focus of NoSQL is:

* Greater scalability in the presence of large datasets or very high write throughput
* A preference for free and OSS over commercial products
* Specialized operations not supported by the relational model
* Alleviation of the restrictiveness of relational schemas

> [Polyglot persistence](https://en.wikipedia.org/wiki/Polyglot_persistence) is the use of different storage technologies in the same application. NoSQL databases can live alongside the relational model. Both can be applied to their strengths.

##### The object-relational mismatch

A common criticism of SQL is that an awkward layer of translation is needed between relational
tables and and objects in application code. Object-relational mapping (ORM) frameworks exist to
paliate this disconnect and to reduce boilerplate, but they can't completely hide the differences.

This mismatch becomes apparent in data models where one-to-many relationships prevail. A JSON or XML
representation (supported by document-oriented databases) can be appropriate for this type of
model (for example a résumé), for the following reasons:

* The JSON representation has better _locality_ (all the necessary data is available without the
  need to extend the query)
* The tree structure in the data becomes apparent in the JSON representation
* The JSON representation lacks a schema (this is cited as both an advantage and a disadvantage)

##### Many-to-one and many-to-many relationships

Database normalization (covered in Part III) is the removal of duplication in a database. Normalized
data is referred to using IDs. If a database is to be normalized, one needs to introduce many-to-one
and/or many-to-many relationships. These don't fit nicely into the document model, since joins (1)
are not needed for one-to-many tree structures and (2) are, if at all, not well supported in
document databases.

##### Are document databases repeating history?

The [hierarchical model](https://en.wikipedia.org/wiki/Hierarchical_database_model) represents data
as a tree of nested records, just like the JSON structure of document databases. The hierarchical
model worked for one-to-many tree structures but made many-to-many relationships difficult and
required manual normalization or the acceptance of denormalization.

These problems of the 1960s and 1970s are similar to the problems developers run into with document
databases today. The proposed solutions back then were the relational model and the network model.

- In the network model, one record could have multiple parents, as opposed to the hierarchical model
  where one record could only have one parent, allowing for many-to-many relationships. The links
  between records were pointers to other records, so one would have to remember an _access path_ to
  traverse from one record to another. Changing access paths was difficult, and it was a
  responsibility of the application developer.
- In the relational model, all the data lays in the open. Query optimizers (in which years of
  research have been spent) do the job of application developers and figure out the access path to
  data from a declarative query.

##### Relational versus document databases today

Points to be taken into account when considering which model to pick are:

- Which will lead to simpler application code? If the data in your application looks like a document
  or a tree of one-to-many relationships, then the document model could be appropriate. If you need
  many-to-many relationships, document databases will force the choice of denormalization or
  emulating joins in application code.
- Do you need a flexible schema? _Schema-on-read_ is akin to dynamic type checking in programming
  languages, and _schema-on-write_ to static type checking. If your data is naturally heterogeneous,
  you may benefit from this. If you need reliable migrations, then pick a relational database.
- Is data locality important to your application? Storage locality could be a performance benefit if
  your application can load the whole document without joins. Document databases could be
  appropriate, but relational databases have ways of grouping related data t ogether: [_multi-table
  index cluster tables_](http://www.dba-oracle.com/oracle_tip_hash_index_cluster_table.htm) on
  Oracle or the [_
  column-family_](https://www.i2tutorials.com/cassandra-tutorial/cassandra-column-family/) concept
  used in Cassandra and HBase.
- Is a hybrid appropriate? Relational database systems support JSON and XML columns, and some
  document databases support relational-like joins and automatic resolution of document references.
  This trend of hybrid databases is likely to become more prevalent in the future.

#### Query languages for data

Query languages can be declarative or imperative:

- Declarative languages resemble relational algebra and are more limited in functionality. This
  allows for the database to apply optimizations more freely.
- Imperative languages encode more knowledge about the querying approach and are therefore harder to
  optimize.

SQL is a declarative query language. Another example of a declarative query language is CSS, which
can be contrasted with manipulating styles imperatively in JavaScript.

##### MapReduce querying

MapReduce was popularized by Google as a way to process large amounts of data in bulk across many
machines. MongoDB and CouchDB (NoSQL datastores) implement a limited form of MapReduce.

MapReduce takes the `map` and `reduce` functions from functional programming languages. In the NoSQL
implementations, the benefit is being able to run JavaScript in the middle of a query.

Some SQL databases can be extended with JavaScript functions, too, such as PostgresDB. Nothing in
SQL restricts it from running on multiple machines, either.

#### Graph-like data models

Graph-like data models are appropriate for applications with many-to-many relationships. A graph
consists of _vertices_ and _edges_.

##### Graph queries in SQL

If we try to model a graph in a relational database we'd end up with a table for edges and another
for vertices. Querying such a database would be difficult with SQL because the number of joins
needed to get to a certain piece of data is not fixed in advance (unlike in the relational model)
. [_Recursive common table expressions_](https://www.postgresql.org/docs/9.1/queries-with.html) can
be used to do this.

##### Types of graphs I: property graphs

Vertices in property graphs have a unique identifier, a set of outgoing edges, a set of incoming
edges, and a collection of properties.

Each edge consists of a unique identifier, the tail vertex (where the edge starts), the head
vertex (where he edge ends), a label that describes the relationship, and a collection of
properties.

Some things are difficult to model in a relational schema but become more natural in property
graphs, for example the regional structures in different countries (counties and states in the USA
versus _départements_ and _régions_ in France). These relationships can be modeled with an "within"
edge.

The [Cypher query language](https://neo4j.com/developer/cypher/) is a declarative query language for
graph databases, and it was built for the Neo4j graph database.

##### Types of graphs II: triple-stores

Triple-stores are equivalent to property graphs but use different words. Information is stored in
three-part statements: subject, predicate and object (Lucy, is, a person).

The [SparQL query language](https://www.w3.org/TR/rdf-sparql-query/) is a query language for
triple-stores that uses [the RDF data model](https://www.w3.org/RDF/).

##### The foundation: Datalog

[Datalog](https://en.wikipedia.org/wiki/Datalog) is a syntactical subset
of [Prolog](https://en.wikipedia.org/wiki/Prolog) and has been studied extensively since the 1980s.
It generalizes triple-stores, writing triples as `predicate(subject, object)`.

### Chapter 3: storage engines

### Chapter 4: serialization and schema evolution

## Part 2: Distributed Systems

### Chapter 5: replication

### Chapter 6: partitioning/sharding

### Chapter 7: transactions

### Chapter 8: problems of distributed systems

### Chapter 9: achieving consistency and consensus in distributed systems

## Part 3: Derived Datasets

### Chapter 10: batch processing

### Chapter 11: stream processing

### Chapter 12: putting it all together
