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

Just like databases, queues and caches can be thought of as data systems (because the line between each is blurred in today's ecosystem), we can also think of our own applications as special-purpose data systems built from generic components that solve more abstract problems.

#### Reliability

A _fault_ is a deviation in the functioning a component from its spec. A _failure_ is a complete service interruption to the user.

Fault tolerance is best thought of as the design of a system such that the probability of a fault causing a failure becomes low: building a reliable system from unreliable parts. Designing a system without faults is not possible.

It's generally preferable to tolerate fault than to try to prevent them, except in the case of security issues, where the event of a system being compromised cannot be undone.

- Hardware faults: machines and components have a mean time to failure (MTTF). The industry has evolved from designing fault tolerance of components (hot-swappable CPUs, generators for backup power) to additionally designing systems that tolerate the loss of entire machines through software.
- Software faults: they usually cause more problems than hardware faults because they're present horizontally across the system. They're harder to predict and usually lie dormant until a certain set of circumstances occur simultaneously.
- Human error: systems can be designed, decoupled, monitored and tested so that opportunities for human error are as thin as possible. Humans can be trained.

#### Scalability

Scalability describes a system's ability to cope with increased load. It must be described in a non-binary way. A system may scale up to (for example) 10,000 concurrent users but not 1 million.

Load parameters are used to describe system load. With Twitter as an example, important parameters are tweets posted per second, home timeline views per second, and the distribution of followers per user (because each user's home timeline loads the tweets of all its followers).

Armed with the ability to describe load parameters, we can now describe the performance of a system in its terms: how is performance affected when a specific load parameter increases and the system resources remain unchanged? How do we need to change system resources to keep performance the same as a specific load parameter increases?

_Response time_ is the amount of time that passes since a client produces a request and the time they get a response back. _Latency_ is the amount of time the request is latent, waiting to be handled.

Response time must be thought of as a distribution and not as a single number, because response time changes on every request, even if the request remains the same.

[Percentiles](https://en.wikipedia.org/wiki/Percentile) are the most common way to describe a distribution of response times. The 95th, 99th and 99.9th percentiles are common, also referred to as p95, p99 and p999.

Percentiles are used in SLOs and SLAs (service-level objectives and service-level agreements) between customers and service providers to define the expected availability and performance of a service.

Head-of-line blocking happens when one slow request causes the slow handling of subsequent requests. This is especially important in services where one user request may trigger multiple calls to backend services. Tail (i.e. high-percentile) latency amplification is a term used to describe a slow-down of multiple user requests due to one such backend service call performing poorly.

To cope with increased load, architectures must be reconsidered (usually) on every order-of-magnitude load increase. One design will not fit all scalability needs.

Designs can _scale up_, or vertically, by moving to a more powerful machine, or _scale out_, or horizontally, by distributing load across more machines. Single-machine architectures are simpler, but a hybrid approach is usually good.

_Elastic_ systems add computing resources autonomously and automatically, but other systems must be scaled manually, and in turn remain simpler.

#### Maintainability

_Operability_ is vital: [good operations can often work around the limitations of bad (or incomplete) software, but good software cannot run reliably with bad operations](https://blog.empathybox.com/post/19574936361/getting-real-about-distributed-system-reliability). To make operations easier, software can provide runtime visibility, support for automation, documentation, good default behavior and predictability, and it can avoid depending on an individual machine.

_Simplicity_ must be attempted with the use of abstractions.

_Evolvability_ can be aided by good organizational processes.

### Chapter 2: data models and query languages

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
