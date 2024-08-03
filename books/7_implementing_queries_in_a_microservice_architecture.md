# 7. Implementing queries in a microservice architecture
- api composition pattern: simplest approach. use whenever possible
- CQRS(Command Query Responsibility Segregation) pattern: powerful but complex

## api composition pattern
- roles
    - api composer
        - queries provider service
        - may be a client / web application / service(api gateway, BFF)
    - provider service: owns and returns data

- whether to use this pattern depends on
    - how the data is partitioned
    - capabilities of the APIs exposed by the data owner service
    - the capabilities of the database
    - ex) if large join is needed -> inefficient

- design issues
    - which component would be a api composer?
        - option1: client service
            - not practical for client outside the firewall or in slow network
        - option2: apigateway
        - option3: api composer as a standalone service
            - choose this if a query is used in multiple services,
            with logic too complex to be in api gateway
    - how to write efficient aggregation logic
        - use reactive programming model(parallel operation)

- benefits and drawbacks
    - benefit
        - simple and intuitive
    - drawbacks
        - increased overhead
            - traditional way 1 request -> multiple requests with this pattern
        - risk of reduced availability
            - with more participants in the operation -> decrease in availability
        - lack of transactional data consistency

## CQRS pattern

- limit of api composition pattern
    - provider may not supports specific options like filter, sorting \
        -> excessive network traffic
    - api composer should do large join \
        -> inefficient
- a challenging single service query
    - data owner service may have DB not good for query
    - if use different database, syncing with replica is hard
    - a service may get too much responsibilities

### CQRS
- maintain read only view(s)
- persistent data model
    - command: create, update, and delete
    - query: queries(get). should synchronize data with command-side model by events
- query only service
    - has api consisting of only query operations
    - good for the separation of concern
- benefits
    - efficient implementation of queries in msa 
    - efficient implementation of diverse queries
        - can use diverse datastore(good for specific purpose)
    - enables querying in event sourcing-based application
    - separation of concern
- drawbacks
    - complex architecture
    - dealing with the replication lag
        - problem: must avoid exposing potential data inconsistency to users
        - solution1: return data with version
        - solution2: in ui application, use local model
            - drawback: ui code must duplicate server-side code

## designing CQRS view
- 1\. choose datastore ans schema
    - handle view, and update from event
    - nosql is often good choice
    - points to think
        - if need pk based lookup of json  \
        -> use document store(mongoDB, DynamoDB) or key value store(redis)
        - if need query based lookup of json object  \
        -> use document store(mongoDB, DynamoDB)
        - if need text queries \
        -> use text search engine(ElasticSearch)
        - if need graph queries \
        -> use graph database(Neo4j)
        - if need traditional SQL repoting/BI \
        -> use RDBMS
- 2\. data access module design
    - handle concurrency. \
      a view may subscribe to events from multiple aggregate.  \
      -> simultaneous update of the same record may occur
    - idempotent event handler
        - non-idempotent event handler must discard duplicate
    - enable a client application to use an eventually consistent view
        - 1\. command-side operation returns a token
        - 2\. client queries with that token.
        - 3\. if the query-side is not updated -> return error
- 3\. adding and updating CQRS views
    - must be able to handle old events
    - update may be expensive
        - update by two phase update(snapshot + events)