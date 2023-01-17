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

Solution: Redundancy, store data into multiple, redundant computers\

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

* Scalability
  Partitioning can be used to achieve scalability:
  
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


  
