# Distributed Systems

## Intro
> “A distributed system is a system whose components are located on different networked computers, which communicate and coordinate their actions by passing messages to one another.”

Assume a node is a machine then

<img width="625" alt="image" src="https://user-images.githubusercontent.com/24993672/212600672-51f70759-e0e1-408a-9c3e-f70e467a4512.png">

<img width="452" alt="image" src="https://user-images.githubusercontent.com/24993672/212601188-8abb0c89-b611-4991-8054-05f9e0bf3860.png">

__Performance__

> Performance is the degree to which a software system or component meets its objectives for timeliness

Solution: Single high-end computer -> Two or more low-spec computers 

__Scalability__

> Scalability is the capability of a system, network, or process to handle a growing amount of work, or its potential to be enlarged to accommodate that growth

Solution: If use distributed system, can split and store the data in multiple computers and distribute the work

__Availability__

> Availability is the probability of a system to work as required, when required, during a mission

Solution: Redundancy, store data into multiple, redundant computers

In summary, distributed system is the solution for all above

### Fallacies

<img width="698" alt="image" src="https://user-images.githubusercontent.com/24993672/212602782-e4f99b19-9952-4cd6-b84c-1ccc8f7a7e1c.png">

### Properties

<img width="592" alt="image" src="https://user-images.githubusercontent.com/24993672/212604106-0f25b342-7ddc-4985-a488-aedda79b3cdc.png">

* Network Asynchrony
 
  It is a property of communication networks that cannot provide strong guarantees around delivering events, e.g., a maximum amount of time a message requires for delivery.
* Partial Failures

  These are the cases where only some components of a distributed system fail.
* Concurrency

  It is the execution of multiple computations at the same time, and potentially on the same piece of data.

### Measures of Correctness 

* Safety Property 
* Liveness Property

### System Models

* Key questions for a distributed system:
  - How the nodes of a distributed system interact with each other
  - How the nodes of a distributed system can fail

* Categories of Distributed systems:
  - __Synchronous systems__: It is one where each node has an accurate clock, and there is a known upper bound on the message transmission delay and processing time. All nodes run in lock-step.
  - __Asynchronous systems__: It is one where there is no fixed upper bound on how long it takes for a node to deliver a message, or how much time elapses between consecutive steps of a node. All nodes run at independent rates.

### Types of Failures
<img width="503" alt="image" src="https://user-images.githubusercontent.com/24993672/212606285-9687ee28-efc0-4a13-8eb5-0aa16a93ba27.png">
* Fail-stop: A node halts and remains halted permanently. Other nodes can detect that the node has failed
* Crash: A node halts, but silently. So, other nodes may not be able to detect this state. 
* Omission: A node fails to respond to incoming requests.
* Byzantine: A node exhibits arbitrary behavior: it may transmit arbitrary messages at arbitrary times, take incorrect steps, or stop.

### Exactly-Once Semantics

As network is not reliable, messages might get lost and nodes may deliver messages multiple times because the sender can't know what really happens. \
__Approaches to avoid multiple deliveries of a message__
* Idempotent operations approach: the operation we apply multiple times has the same result with the initial application. for ex: set (not allow remove), counter-ex: increase counter by one each time
* De-duplication approach: Sender gives every message has a unique identifier, and the receiver will know if the received message is duplicate by recognizing its id.

__Delivery vs. Processing__\
* Delivery: arrival of the message at the destination node at the hardware level
* Processing: handling of the message from the software application layer of the node

So important and possible is exactly-Once for Processing, but not for Delivery\
__Other Delivery Semantics__: at-most-once -- send every message only once, at-least-once -- send a message continuously until get an acknowledgement from recipient

### Failure

Reason: The asynchronous nature of the network in a distributed system makes it hard to differentiate between a crashed node and a node that is just really slow to respond to requests.\
One mechanism to detect failure: __Timeouts__\
Trade-off between small or large timeout value: too small -> false positives, too large -> waiting time\
Failure Detector Properties:
* Completeness: the percentage of crashed nodes a failure detector identifies in a certain period
* Accuracy: number of mistakes that the detector makes in a certain period

There is no perfect failure detector

### Stateless and Stateful Systems

Stateless distributed systems are much easier to design, build and scale. 
Stateful systems present many more challenges as different nodes can hold different pieces of data which require additional work. They need to direct traffic to the right place and ensure each instance is in sync with the others.

## Concepts and Theorems

### Partitioning

__Scalability__\
  Partitioning can improve the scalability and performance of a system by distributing data and request load to multiple nodes:
  
<img width="561" alt="image" src="https://user-images.githubusercontent.com/24993672/212787819-39c794c7-6dfe-47f6-bc7c-0fa0485d3ae4.png">

  - Vertical Partitioning: join operation, normalization. limitation -- join operation may be less efficient and need to access data from multiple nodes. Data modeling Practice. 
  - Horizontal Partitioning (or sharding): alphabetical split, limitation -- search may need to access multiple nodes, potential for loss of transactional semantics, harder to perform atomic operations. A common feature of distributed databases.

### Algorithms for Horizontal Partitioning

* Range Partitioning

  Split a dataset into ranges according to the value of a specific attribute. Then store each range in a separate node. The system should store a list of all ranges and map which node stores a specific range. Example: alphabetical split.\
  Pros: Simplicity and ease of implementation, range queries using partitioning key value, good performance for range queries when the queried range is small and resides in a single node, adjusting ranges (re-partitioning) easier and more efficient (exchanges data only between two nodes)\
  Cons: can't query range using keys other than partitioning key, bad performance for range queries when the queried range is bid and resides in multiple nodes, uneven distribution of the traffic or data may cause some nodes to overload (for ex, for alphabetical split, some letters may appear as the initial letters more frequently than others)\
  Some systems that leverage this technique: Google's BigTable, Apache HBase. 
  
* Hash Partitioning

  Apply a hash function to a specific attribute of each row. This results in a number that determines which partition (node) this row belongs to. \
  Mapping Process: Assume we have n number of nodes, and try to identify which node locates a record with a value s, we'll calculate it with the fomula __hash(s) mod n__.\
  The mapping will take place both when we write a new record or receive a request to find a record. And if any node failed, the re-mapping and reassignment is needed.\
  Pros: No need to store and maintain the mapping because it can calculate the partitioning mapping at runtime -> great for both data storage and performance (no additional requests to find the mapping), most likely the hash function will uniformly distribute the data across our system's nodes and prevent overloading of some nodes\
  Cons: Can't perform range queries, adding or removing nodes causes repartition -> significant data movement
  
* Consistent Hashing

  It is similar to Hash Partitioning, but solves the increased data movement problem caused by hash partitioning. \
  Each node in the system is randomly assigned an integer in a range of [0, L], the range is called __Ring__. Then the system uses a record with an attribute value __s__ as a partitioning key to locating the node after the point __hash(s) mod L__ in the ring. So when a new node enters the ring, it receives data only from the next node in the ring, all other nodes don't need to exchange any more data. It is similar when a node leaves the ring. 
  <img width="698" alt="image" src="https://user-images.githubusercontent.com/24993672/212797669-3f46bd9b-9d12-4c39-af61-6925a09b6f73.png">
  
  <img width="619" alt="image" src="https://user-images.githubusercontent.com/24993672/212797716-9e42a029-9223-4f19-8e5e-9d3a0d735b9d.png">
  
  <img width="695" alt="image" src="https://user-images.githubusercontent.com/24993672/212797745-eaf5b43c-cee0-4dd6-9a87-f4b59bbce0d9.png">

  <img width="634" alt="image" src="https://user-images.githubusercontent.com/24993672/212797774-db6185bc-345a-4ad4-a7e8-f19e5e83bdae.png">

  <img width="712" alt="image" src="https://user-images.githubusercontent.com/24993672/212797792-9c353c9a-2da5-4d78-bd96-bcb7dcc2db97.png">

  Pros: Comparing to hash partitioning, it reduced data movement when add or remove nodes\
  Cons: Potential for the data's non-uniform distributetion because of the random assignment of nodes in the rings, potential for more imbalanced data distribution when add or remove nodes. \
  However, We can use __Virtual nodes__ (assign each physical node multiple locations in the ring) to mitigate these issues. \
  Some systems that leverage this technique: Apache Cassandra

### Replication

__Availability__ (the ability of the system to remain functional despite failures in parts of it)\
  Replication is used to increase availability. Replication includes storing the same piece of data in multiple nodes (called replicas) so if any one crashes, data is not lost and requests can be served from the other nodes meanwhile. \
  Two main strategies for replication:
  * Pessimistic replication: try to guarantee that all the replicas are identical from the beginning
  * Optimistic replication (lazy replication): allows the different replicas to diverge. They will converge again if the system doesn't receive any updates or enter a quiesced state for a period of time 
  
### Single-Master Replication Algorithm

  Alias, __primary-backup replication__.
  > It is a technique where we designate a single node amongst the replicas as the leader (or primary) that receives all the updates. Remaining replicas are followers (or secondaries) which only handle read requests. \
  Leader receives updates and executes them locally and __propagates__ the updates to the followers. This ensures the consistent view of the data for all replicas. 
  <img width="405" alt="image" src="https://user-images.githubusercontent.com/24993672/212803284-05c67620-eede-4f3f-a712-ca2e1169fca8.png">
  
#### Techniques for propagating updates
  * Synchronous replication
  
    The node signals client that the update is completed only if the node received acknowledgements of completed updates from all other replicas. \
    Pro: Durability. (If the leader crashes right after acknowledgement, the update is still kept.) \
    Con: Writing requests are slower. (it has to wait for responses from all other replicas)
  * Asynchronous replication
  
    The node signals client that update is completed as soon as it performs the update itself in its local storage, without waiting for responses from the other replicas.\
    Pro: Increase performance for write requests.
    Con: reduced consistency and decreased durability 
    
#### Pros and Cons of single-master replication

  Pros: simple, easier to support transacional operations because concurrent operatiions serialized in the leader node, __scalable for read-heavy workloads__ \
  Cons: __not scalable for write-heavy workload__, impose the trade-off between performance, durability and consistency, too many listening followers may create bottleneck in the network bandwidth of the leader node, failing over to followers from crashed leader is slow -> downtime and risk of errors\
  Examples of applications: PostgreSQL, MySQL
  
#### Failover
  
  * Manual approach: operator selects the new leader 
  * Automated approach: the followers detect periodically if the leader is healthy and if not then elect a new leader. Faster but risky if the followers get incorrect state of leader
  
### Multi-Master Replication Algorithm 

  Alias, __multi-primary replication__
  > It is an alternative replication technique that favors higher availability and performance over data consistency. In it, all replicas are equal and can accept write requests and are responsible for propagating the data modification to the rest of the group.

  Write requests are concurrently handled by all the nodes and they may have conflict (disagree on the order for some requests) caused by latency of the propagation requests between nodes.\
  Conflict Resolution: \
  (eagerly: resolved during write operation, lazily: resolved after write operation, i.e., subsequent read operations)
  * Exposing conflict resolution to the clients (client selects the version, i.e., shopping cart app)
  * Last-write-wins conflict resolution (each version is tagged with a timestamp using local clock, always select the version with the latest timestamp. but because there is no global notion of time, it may make error)
  * Causality tracking algorithms (use an algorithm to keep track of causal relationships between different requests, but for requests that are actually not causally related i.e. concurrent, it is hard to resolve conflict)

### Quorums in Distributed Systems

* Problem in Synchronous Replicatioin

  Availability is low for write operations, because the failure of a single node makes the system unable to process writes until the node recovers.
  
* General solution

  Reverse strategy: write data only to the node that is responsible for processing a write operation, but process read operations by reading from all the nodes and returning the latest value. \
  This increases write availability but decreases read availability.
  
* Mechanism that trade-off above with balance: __Quorum-based voting protocol__

  <img width="506" alt="image" src="https://user-images.githubusercontent.com/24993672/213953692-efeb7439-072f-453e-9a3c-78baef4d0d54.png">

  First rule gurantees that a data item is not read and written by two operations concurrently.\
  Second rule ensures that at least one node receives both of the two write operations and imposes an order on them. \
  The concept of a quorum is used extensively in other areas, like distributed transactions or consensus protocols.
  
### Safety Guarantees in Distributed Systems

  <img width="419" alt="image" src="https://user-images.githubusercontent.com/24993672/215363189-47e01406-9a09-427b-b99f-2a1b0493235d.png">
  
  Corresponding to the difficulties caused by the following properties:\
  <img width="580" alt="image" src="https://user-images.githubusercontent.com/24993672/215363415-0bfa02cc-5848-479b-b89a-ed4abfdfd054.png">


  * Atomicity: challenging is Partial failures
  * Consistency: challenging is Network Asynchrony 
  * Isolation: challenging is inherent Concurrency

### ACID (Atomicity, Consistency, Isolation, Durability) in traditional db transactions 
  > __Atomicity__ guarantees that a transaction that comprises multiple operations is treated as a single unit. So, the operation is either executed to all the nodes or none.\
  > __Consistency__ guarantees that a transaction only transitions the database from one valid state to another valid state, while maintaining any database invariants. For ex, one record in table A with reference to record in table B thru fk, db will prevent a transaction from deleting a record from A unless the refered record in B is already deleted.\
  > __Isolation__ guarantees that even though transactions might run concurrently and have data dependencies, the result is as if one of them was executed at a time and there was no interference between them.\
  > __Durability__ guarantees that once a transaction is committed, it remains committed even in the case of failure. In the context of distributed systems, this means that transactions need to be durably stored in multiple nodes.
    
### CAP Theorem (Consistency, Availability, Partition Tolerance)

  > __Consistency__ means that every successful read request receives the result of the most recent write request. \
  > __Availability__ means that every request receives a non-error response, without any guarantees on whether it reflects the most recent write request. \
  > __Partition tolerance__ means that the system can continue to operate despite an arbitrary number of messages being dropped by the network between nodes due to a __network partition__. Trade-off between consistency or availability. 

  Use Case: A user performs a write and then a read. Assume that a different node processes each operation (network partition), then: 1. it can fail one of the operations and break the availability property, or 2. it can process both the operations, which will return a stale value from the read and break the consistency property
  
  
  Choice between AP, CP only happens during a network partition. \
  <img width="473" alt="image" src="https://user-images.githubusercontent.com/24993672/215365458-fc2c0eda-53dc-4d91-aa5e-7bc0b15e819e.png">
  
  
  __PACELC Theorem__
  
  When no network partition, both availability and consistency can be satisfied. But there is a trade-off between latency and consistency:\
  To guarantee data consistency, the system needs to delay write operations until the data has been propagated across the ststem, thus -> latency hit. For ex, single-master replication scheme with synchronous replication (consistency > latency). \
  In addition to CP, AP, new categories: EL, EC.
  
#### Consistency Models

  * Linearizability: 
    Operations appear to be instantaneous to the external client. That said, once an operation is complete and acknowledgement is delivered to the client, it is immediately visible to all other clients. Usage: synchronous replication. Can build logic like mutexes, semaphores, counters that needs stronge consistency models.
  * Sequential Consistency:
    Weaker consistency Model. Operations are allowed to take effect before their invocation or after their completion. Operations from different clients have to be seen in the same order by all other clients. Usage: We expect that posts from a single friend and comments in a post to be displayed in the right order.
  * Causal Consistency:
    Weaker consistency Model. Requires that only operations that are causally related need to be seen in the same order by all the nodes. Usage: display comments out of chronological order so every comment is displayed after the comment it replies to.
  * Eventual Consistency:
    One of the weakest consistency model. Usage: As long as the system eventually arrives at a stable state and thus read ops will return the same result, reads don't need to return the latest write. Inconsistencies can be resolved at the application level. 
    
### Isolation Levels and Anomalies 

  __Isolation Levels__
  * Serializability: two transactions executed concurrently should give the same result as though executed sequentially
  * Repeatable read: the data once read by a transaction will not change thru-out its course
  * Snapshot isolation: all reads made in a transaction see a consistent snapshot of the db from the point it started and till the transaction commits successfully if no other transaction has updated the same data since the snapshot
  * Read committed: it doesn't allow transactions to read data that has not yet been committed by another transaction
  * Read uncommitted: it is the lowest isolation level and allows the transaction to read uncommitted data by other transactions

  __Anomalies__
  * Dirty writes: a transaction overwrite a value that was written but not committed yet. Violate integrity constraints, and make rollback to previous image impossible
  * Dirty reads: Read a value that has been written but not yet committed.
  * (Fuzzy) non-repeatable reads: A value is retrieved twice during a transaction but the value is different.
  * Phantom reads: a transaction does a predicate-based read, and another transaction writes or removes a data item matched by that predicate while the first is still in flight.
  * Lost updates: two transactions read the same value and then try to update it to two different values.
  <img width="641" alt="image" src="https://user-images.githubusercontent.com/24993672/229324794-764826ef-8b78-4d7e-a4d0-2a5208e4b4ef.png">

  * Read skew: there are integrity constraints between two data items that seem to be violated because a transaction can only see partial results of another transaction 
  <img width="629" alt="image" src="https://user-images.githubusercontent.com/24993672/229324804-9c713883-b47e-4aa4-bf8d-0b82324908f3.png">

  * Write skew: occurs when two transactions read the same data, but then modify disjoint sets of data
  <img width="629" alt="image" src="https://user-images.githubusercontent.com/24993672/229324808-8f8472bc-e934-46d8-ade5-1f0853c2a8f0.png">

### Prevention of Anomalies

  * Serializable isolation level:  prevents all of the anomalies
  
  <img width="920" alt="image" src="https://user-images.githubusercontent.com/24993672/229325081-ec1ae560-e341-4699-89a0-c5c4b55633d7.png">

### Similarities and Differences between consistency models & isolation levels
  
  <img width="496" alt="image" src="https://user-images.githubusercontent.com/24993672/229325184-1c26fb1b-8afa-4fa7-b5b7-533cd1a73547.png">

  * Similarities 
    Both are essential constructs that allow us to express:

      - Which executions are possible
      - Which executions are not possible
      
    stricter and fewer executions -> increased safety at the cost of reduced performance and availability 
    
    * Differences
      - Consistency models are applied to single-object operations (e.g. read/write to a single register), while isolation levels are applied to multi-object operations (e.g. read and write from/to multiple rows in a table within a transaction).
      - Linearizability provides real-time guarantees, while serializability does not.

    * Real-time guarantees' importance 
    
    <img width="442" alt="image" src="https://user-images.githubusercontent.com/24993672/229325697-af06ab81-2947-41ca-b735-625d6ac42a1f.png">
   
    So need __strict serializability__: a model that is a combination of linearizability and serializability 
    
    * Hierarchy tree
    
    <img width="708" alt="image" src="https://user-images.githubusercontent.com/24993672/229325839-5608b35f-fbdd-41ab-8b48-db470e712291.png">

  
  
  
  
    

  
  
  
  
