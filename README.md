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

This chapter discusses how databases store and retrieve data when developers query it. The
motivation for an application developer to learn this is that knowing database internals at a basic
level can help select a database and fine-tune it for any particular workload that an application
may require.

Two families of storage engines are presented: _log-structured_ and _page-oriented_ storage engines.

#### Data structures that power your database

An example database engine implemented with just two Bash scripts `db_set` and `db_get` is used for
illustration purposes. This engine appends to a log-structured file when `db_set` is called with a
key-value pair, and looks up a value when `db_get` is called with a key.

Appending to it is cheap, but looking up a value is expensive: the engine needs to scan the whole
file to do this every time we call `db_get`.

In order to look up values in our database efficiently, we need an _index_. Indexes are separate,
auxiliary structures that act as sign posts, speeding up reads. But there's a trade-off: since
indexes need to be updated when new data gets written to the database, the presence of indexes may
also slow down writes.

##### Hash indexes

A hash index resembles a hash map. With the example `db_get`, `db_set` database engine described
above, our hash index could live in memory and contain every key in the database, mapped to a byte
offset in the database file. The [BitCask storage engine](https://riak.com/assets/bitcask-intro.pdf)
uses this indexing approach.

To avoid running out of disk space as the database file grows, it can be segmented using a specific
segment size. Once separated in segments, the database file can go through the processes of _
compaction_ (the removal of entries for each key if they are not the most recent) and
_merging_ (concatenating compacted segments together to make them fit into the configured segment
size).

Compaction and merging can happen in new copies of old segments to allow reading from those old
segments in the meantime. The redundant old segments can be removed afterward.

Although the design is simple, implementation details matter: the file format (usually binary, not
text), deletions (append-only data files use a _tombstone_ entry to represent the deletion of a key)
, crash recovery (persisting the in-memory hash maps to disk), partially-written records (detected
using, e.g., checksums), and concurrency control (usually avoided by only having one writer thread).

Why not update the file in place instead of writing a new entry every time?

- Sequential writes are faster than random writes.
- Concurrency and crash recovery are simpler (no overwrites).
- Merging avoids fragmentation of the data file.

Some disadvantages: the hash index must fit in memory, and querying for a range of keys is
inefficient.

##### SSTables and LSM-trees

Our segment files from the previous section are sorted in the order in which they were written. An
SSTable (Sorted String Table) has the additional requirement that its entries be sorted by key.
Adding a new entry is no longer immediate due to this requirement (writing to an SSTable is covered
later).

Advantages:

- Merging segments is easier (with [merge sort](https://en.wikipedia.org/wiki/Merge_sort):
  look at the nth key of every segment file, take the one with the lowest key into the new merged
  segment, repeat).
- To find a key in the file efficiently, you no longer need a hash table with all keys (since the
  keys are sorted, a sparse index suffices: you can use known offsets like the beginning entries in
  a segment to start looking at a certain location and stop looking at another).
- Records between two keys of the sparse index can be compressed, since read queries must scan over
  all the records in this block anyway.

[B-trees](#b-trees) can be used to maintain a sorted structure on disk, but it is easier in memory,
where red-black trees or AVL trees can be used.

> Note from @fnune: this is because B-trees tend to have a lower height than AVL or red-black trees, and so require a lower amount of disk reads to find a specific key.

Once this tree structure (also called a _memtable_) grows too big, it can be written to disk using
an SSTable. Merging and compaction can be scheduled routinely on segment files. To serve a read
request, read the memtable first, then read segment files from newer to older. This structure is
also called an [LSM-tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree).

To prevent data loss in the event of a crash (due to the memtable disappearing) an append-only log
can be kept on disk to mimic the memtable, but unsorted, to recover from it.

To optimize access of keys that don't exist in the database,
a [Bloom filter](https://en.wikipedia.org/wiki/Bloom_filter) may be used.

Common compaction strategies are _size-tiered_ compaction (newer, smaller segments merged into
older, larger segments) and _leveled_ compaction (older data is moved into separate "levels" for
parts of the full key range).

##### B-trees

The indexing structures described earlier organize data into variable-size _segments_. B-trees are
organized into fixed-size _blocks_ or _pages_, which are read one at a time. This matches the
underlying hardware closely.

In B-trees, each page contains a continuous range of keys pointing to child pages, and eventually
leaf pages contain the value for each key.

For example, a B-Tree with a branching factor `m` of 5 indexing items 1 to 100:

- Root page: `[ref C1, 20, ref C2, 40, ref C3, 60, ref C4, 80, ref C5, 100]`
- Page C1: `[ref C1.1, 4, ref C1.2, 8, ref C1.3, 12, ref C1.4, 16, ref C1.5, 20]`

To look up number 16 you would find `ref C1` on the root page, `ref C1.4` on page `C1`, and
eventually find 16.

The branching factor in B-trees used in databases is normally several hundred. If a new value were
to be inserted that would make the page exceed the branching factor, it would be split into two
half-full pages. The parent page would be updated accordingly. This keeps B-trees balanced.

Pages in B-trees are overwritten when they're updated, but their location remains unchanged (and so
do references pointing to them).

Because pages are overwritten in place, a _write-ahead log_ (or WAL) is used to make the database
resilient to crashes. Before writing to a B-tree, databases always write to the WAL. Some databases
optimize this away by using a copy-on-write scheme to modify pages.

To speed up B-trees, key ranges can be abbreviated to save space (since the magnitude of each page
is known from traversing each parent), leaf pages can be positioned sequentially on disk, and
sibling pointers can be added to pages (to avoid having to look up parents in range queries).

##### Comparing B-trees and LSM-trees

LSM-trees are thought to be faster for writes (because B-trees write every piece of data once to the
WAL and once to the corresponding page), and B-trees faster for reads (because LSM-trees need to
look up data in the memtable first, and then in segment files by order).

LSM-trees sometimes have lower _write amplification_ (one write resulting in multiple disk writes)
than B-trees, and so may sustain higher write throughput. They can be compressed better (B-trees
have up to half of each page empty). However, the compaction process can interfere with read and
write performance due to limited disk bandwidth.

B-trees favor the locking mechanisms of database implementations: each key exists only once in the
tree (as opposed to LSM-trees, where keys are replicated across memtable and segment files), so the
lock can be attached directly to the tree.

##### Other indexing structures

_Secondary indexes_ are useful for performing joins efficiently. In them, a value may be indexed
twice, each time with a different key.

In some situations, storing a row value directly in the index (this is known as a _clustered index_)
can be beneficial if the extra hop from the index to the main row is intolerable.

> Note from @fnune: in clustered indexes, the index values may actually _be the table_, they may be not a secondary copy of the table, but the main one. E.g. [Microsoft SQL Server documentation](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described?view=sql-server-ver15) says "there can be only one clustered index per table, because the data rows themselves can be stored in only one order".

Data rows stored in an unordered manner (e.g. not in a clustered index) form a structure called a
heap.

_Multi-column indexes_ are used to store (among other things) geospatial data, which B-tree or
LSM-tree indexes cannot access efficiently, by mapping latitude and longitude to a single number and
then using a regular B-tree index to query that. E-commerce applications may use multi-column
indexes to query for red t-shirts.

_Full-text and fuzzy indexes_ may be used to query for a definition using synonyms of a word, or
searching inside a
certain [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance).

#### Transaction processing or analytics?

##### Data warehousing

##### Stars and snowflakes: schemas for analytics

#### Column-oriented storage

##### Column compression

##### Sort order in column storage

##### Writing to column-oriented storage

##### Aggregation: data cubes and materialized views

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
