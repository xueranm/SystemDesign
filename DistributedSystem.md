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

