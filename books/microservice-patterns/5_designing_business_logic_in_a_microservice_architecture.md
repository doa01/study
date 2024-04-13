# 5. Designing business logic in a microservice architecture

## business logic organization patterns

- inbound adapter: handles request from client
- outbound adapter: invokes other service and application
- adapters
  - REST API adapter: inbound adapter that implements REST API
  - OrderCommandHandler: inbound adapter that consumes command message
  - Database Adapter: outbound adapter that access the database
  - Domain Event publishing adapter: outbound adapter that publishes events to message broker

### transaction script pattern

- script(business logic) is located in service class
- one method for each request/system operation
- access database using dao. pure data object with no behavior

### domain model pattern

#### object oriented design

- business logic consists of an object model, a network of relatively small classes
- benefits
  - easy to understand and maintain(because it is small)
  - easier to test

#### domain driven design

- building blocks

  - entity
    - an object that has persistent identity
    - two entities whose attributes have the same value are still different
  - value object
    - a collection of values
    - two value objects whose attributes have the same value can be used interchangeably
  - factory
    - an object or method that implements object creation logic that is too complex to use constructor.
  - repository
    - an object that provides access to persistent entities and encapsulates the mechanism for accessing the database
  - service
    - an object that implements business logic that doesn't belong in an entity or a value object

- fuzzy boundaries
  - fuzzy boundary makes a problem when updating business object as well as conceptual unclarity

#### aggregates have explicit boundaries

- aggregate
  - a cluster of domain objects within a boundary that can be treated as a unit
  - root entity + entities + value objects
- aggregates are consistency boundary
  - updates are invoked on the aggregate root
  - concurrency is handled by locking the aggregate root

#### aggregate rules

- rule1: reference only the aggregate root
  - root entity is the only part of an aggregate that ca be referenced by outside the aggregate
  - ex) a service loads an aggregate from DB and obtains a reference to the the aggregate root. \
    updates an aggregate by invoking a method on the aggregate root
- rule2: inter-aggregate references must use primary keys
- rule3: one transaction creates or updates one aggregate

### aggregate granularity

- ideally small
  - improve scalability, reduce conflicting updates
- large enough
  - atomic update

## publishing domain events

### what is domain event?

- each property is either primitive value or value object
- has metadata
  - event id, timestamp, who made the change, ...
  - can be in an envelope object that wraps the event object \
     the id of the aggregate may be in the envelope

### event enrichment

- event contain information that consumer needs
- benefit
  - simplifies the consumer
- drawbacks
  - event class changes whenever the requirements of the consumer change
  - in many cases, it is quite clear that which properties need to be in an event

### identifying domain events - event storming

- event storming: workshop format for understanding a complex domain
  - 1\. brainstorm events
  - 2\/. identify event triggers
  - 3\. identify aggregates

### generating and publishing domain events

- aggregate generates the events and return them to the service
  - because aggregates can't use dependency injection
  - implementation1: the return value of an aggregate method to include a list of events
  - implementation2: aggregate root to accumulate events in a field
  - author prefers implementation1
- to reliably publish domain events, use transactional messaging

### consuming domain event

- in Eventuate Tram framework, `DomainEventDispatcher` dispatches domain event to appropriate handler method
