# 3. interprocess communication in a microservice architecture

## interaction styles

### categories

- processed by
  - one-to-one: each client request is processed by one service(orthogonal to IPC technologies)
  - one-to-many: each request is processed by multiple services
- sync/async
  - synchronous: the client expects timely response, and might event block while it waits
  - asynchronous: the client doesn't block, and the response, if any, isn't necessarily sent immediately

|              | one-to-one                                                                    | one-to-many                                     |
| ------------ | ----------------------------------------------------------------------------- | ----------------------------------------------- |
| synchronous  | request/response                                                              | -                                               |
| asynchronous | asynchronous request/response <br/> one-way notifications(reply not expected) | publish/subscribe <br/> publish/async responses |

---

## communicating using the synchronous remote procedure invocation pattern

- basic structure
  - business logic --(invokes)--> proxy-interface(implemented by an RPI proxy adapter class)
  - RPI proxy adapter class makes request
  - the request is handled by an RPI serer adapter class

### using rest

- rest
  - key concept: resource - represents a single business object
  - verb - http method
- rest maturity model
  - level 0 : client makes HTTP POST request to a single endpoint. each request specifies the action and the target
  - level 1 : resource is in the url. method is still POST only
  - level 2: action is in http method
  - level 3: based on HATEOAS(Hypertext As the Engine of Application State) principle. the response of GET request contains the links for performing actions on that resource
- IDL
  - open api specification: most popular
- challenge of fetching multiple resources in a single request
  - one solution: allow the client to get related resources when it gets a resource. ex) query param. `GET /orders/2?expand=consumer`
    - works well with many scenario
    - but may be time consuming to implement
    - others: graphQL, netflix Falcor
  - challenge of mapping operations to http verbs
    - rest api should use PUT to update, but there may be many kinds of updates. ex) cancel, ...
    - update may not be idempotent, which is required for PUT
    - one solution: define sub-resource ex) `PUT /orders/2/cancel`
    - another solution: verb as a url query param ex) `PUT /orders/2?action=cancel`
    - neither is RESTful
    - this led to the popularity of alternatives ex gRPC
- benefits
  - simple and familiar
  - can test http API with browser
  - directly support request/response style communication
  - firewall friendly
  - doesn't require intermediate broker -> simplify architecture
- drawbacks
  - only supports request/response style communication
  - reduce availability. no buffer -> client and server must be running when communicating
  - client must know the urls of the service
  - fetching multiple resources in a single request is challenging
  - sometimes difficult to map multiple operations to http verbs

### using gRPC

- binary message-based protocol
- http/2
- protocol buffet compiler generates client-side stubs, server-side skeleton, in various languages
- IDL: protocol buffers based idl
- service definition is a collection of strongly typed methods.
- supports streaming RPC
- use protocol buffer as message format
  - efficient, compact, binary format
  - tagged format. each field of a message is numbered and has type code
  - recipient can extract fields needed and skip over the field -> supports backward compatibility
- benefits
  - good to design API with various update operation
  - efficient, compact IPC mechanism. especially when exchanging large messages
  - supports bidirectional streaming -> RPI and messaging styles are possible
  - supports client and services of wide range of languages
- drawbacks
  - javascript client needs additional work
  - older firewall might not support http/2
  - synchronous communication
  - partial failure

### handling partial failure using circuit breaker pattern

- prevent cascading of failure
- 2 parts of solutions
  - design RPI proxy to handle unresponsive remote services
  - decide how to recover from a failed remote service
- robust RPI proxies
  - implement these mechanism(from [netflix's article](https://netflixtechblog.com/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a))
  - network timeout
  - limiting the number of outstanding requests from a client to a service
  - circuit breaker pattern: track the number of successful and failed request, and block the request when the error rate exceeds some threshold
- recovering from an unavailable service
  - option1: return an error
  - option2: return default value or cached response

### using service discovery

- two types
  - service and the clients interact directly with the service registry
  - deployment infrastructure handles service discovery
- application level service discovery pattern
  - self registration pattern
    - service instance invokes the service registry's registration API
    - may also support health check url
  - client side discovery pattern
    - client queries the service registry to get a list of service instances
  - benefit: independent from deployment platforms
  - drawbacks: need a library for every language
- platform-provided service discovery pattern
  - built-in service registry and service discovery mechanism from Docker, Kubernetes
  - combination of two patterns
    - 3rd party registration pattern: 3rd party called registrar handles registration
    - server-side discovery pattern: client makes a request to a dns name instead of querying service registry
    - benefit: handled by deployment platform
    - drawback: platform dependent

## communicating using the asynchronous messaging pattern

- typically uses message broker - intermediary between services. or, brokerless architecture
- client makes a request to a service by sending it a message, if reply needed, send a separate message back to the client

### overview of messaging

- composition
  - header
    - collection of name-value pairs
    - metadata that describe the data being sent
    - unique message-id generated by the sender or the messaging infrastructure
    - optional return address for the reply
  - body
    - the data being sent. text or binary format
- kinds
  - document: generic message that contains only data. receiver decides how to interpret
  - command: equivalent of an RPC request. specifies the operation to invoke and its parameters
  - event: something notable occurred

### message channels

- two kinds of channels
  - point-to-point
    - exactly one consumer. one-to-one interaction.
    - ex) command is often sent over point-to-point channel
  - publish-subscribe
    - deliver message to all attached consumers
    - ex) event message
- implementation
  - request/response
    - can block until a reply is received
  - asynchronous request/response
    - doesn't expect the reply
    - can get reply from reply channel
      - reply contains correlation id that has the same value as message identifier
  - one-way notification
    - doesn't send back a reply
  - publish/subscribe
    - publish a message to a channel that is read by multiple consumers
    - domain event is usually represent changes of a domain object
  - publish/async response
    - client publishes a message with reply channel header
    - consumer writes a reply message with correlation id
- creating api specification for a message based service api
  - specify: name of message channel, message type, format
  - no standard for documentation
  - documenting async operations
    - request/async response style
      - command message channel
      - the types and formats of the command message that the service accepts
      - the types and formats of the reply message
    - one-way notification style
      - command message channel
      - the types and formats of the command message that the service accepts
  - documenting published events
    - message channel
    - the types and formats of the event that the service accepts

### with message broker, brokerless messaging

- brokerless messaging

  - popular messaging technology
    - zeroMQ
  - benefits
    - lighter network network traffic, better latency
    - eliminate the possibility of message brokers being performance bottleneck
    - less operational complexity
  - drawbacks
    - services needs to know each other's location
    - reduced availability. both sender and receiver should be available
    - implementing mechanisms like guaranteed delivery is more challenging

- broker-based messaging
  - popular messaging technology
    - activeMQ
    - rabbitMQ
    - apache Kafka
  - factors to consider
    - supported programming languages
    - supported messaging standards
    - messaging orders
    - delivery guarantees
    - persistence
    - durability: can receive the message that were sent while the consumer was disconnected?
    - scalability
    - latency
    - competing consumer
  - benefits
    - loose coupling
    - message buffering
    - flexible communication: supports diverse interaction styles
    - explicit interprocess communications
  - drawbacks
    - potential performance bottleneck
    - potential single point of failure
    - additional operational complexity

### competing receivers and message ordering

- common solution:sharded(partitioned) channels
  - a sharded channel consists of two or more shards. each behaves like a channel
  - sender specifies a shard key in the message header. \
    message broker uses a shard key to assign message to a particular shard/partition
  - message broker assigns each shard to a single logical receiver( a group of instances. ex) consumer group). reassigns shards when receivers start up and shut down

### handling duplicate messages

- exactly-once is costly, instead, mostly at-least-once
- though broker guarantees at-least-once, the client/network/broker may fail and message may be delivered multiple times
- solutions
  - write idempotent message handler
  - track messages and discard duplicates
    - simple solution
      - message id and discard duplicates
      - option 1: records message id in the database as part of the transaction and create/updates business entities
      - option 2: record message id in an application table.(for no sql database)

### transactional messaging

- database update and event publishing should be within a transaction
- traditional solution: distributed transaction <- not good

#### transactional outbox pattern

- database table as a temporary message queue
- MessageRelay: a component that reads the OUTBOX table and publishes the message to a message broker

#### publishing events

- polling publisher pattern
  - simple, but frequent polling may be expensive
  - applicable with NoSQL DB depends on its querying capabilities. querying business entities may not be efficient
- transaction log tailing pattern
  - a transaction log miner can read the transaction log and publish each change as a message to the message broker
  - applicable to both RDBMS, NoSQL DB
  - examples: debezium, linkedin databus, dynamoDB streams, eventuate tram
  - challenge: requires some development effort

#### libraries and frameworks for messaging

- message broker's client library: drawbacks
  - couples business logic that publish messages to the message broker API
  - typically low level
  - usually provider only basic mechanism
- better to use high level library/frameworks

### eliminating synchronous interaction

- one way: replicate data
  - drawback
    - inefficient to replicate large amounts of data
    - how a service data owned by other services
- finish processing after returning a response
  - fot client to know whether the operation was successfully done, either
    - periodically poll
    - or the service send notification message
