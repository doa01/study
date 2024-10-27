# 11. Developing production-ready services

- quality attributes
  - application security
  - service configurability: separate configuration with code
  - observability

## 11.1 secure services

- authentication
- authorization
- audit
- secure interprocess communication

### authentication

- security in traditional monolithic application
  - user login with ID & password
    - client make request to server
    - server returns session token
  - client do further request with this session token
- problem
  - in-memory security context
    - should always be routed to same application instance
  - centralized session
    - should not depend on one database
- how to solve
  - simple: handle authentication in each service
    - problem
      - unauthenticated request enter internal network
      - each service may implement authentication differently
  - better way: handle authentication in API gateway
    - client request login with credential -> API gateway authenticates and return security token
    - client make further request with security token -> API gateway validates and forwards if okay

### authorization

- where

  - API gateway

    - drawback
      - coupling API gateway with service
      - only url path based check is available
        -not practical

  - better way: service

- security token

  - opaque token
    - usually uid
    - need to call security service to validate and retrieve information
  - transparent token
    - token contains information
    - standard: JWT
      - self-contained, irrevocable -> use short-lived jwt.

- oauth2.0

  - key concept
    - authorization server
    - access token
    - refresh token: long-lived yet revocable token that a client uses to obtain a new access token
    - resource server: a server that use access token to authorize access
    - client
  - process
    - if the access token has expired or is about to expire,
      the API Gateway obtains a new access token by making an OAuth 2.0 Refresh Grant request to the authorization server
  - benefit
    - proven security standard

- key ideas
  - api gateway is responsible for authenticating client
  - api gateway and the service use a transparent token(ex. JWT)
  - service use the token to obtain the principal's identity and roles

## 11.2 configurable services

- push model

  - deployment infrastructure push configuration properties to service
  - format: environment variable, file, ..
  - infra and server must agree on how
  - benefit: effective and widely used
  - limitation: reconfiguration may be hard or impossible without restart
    (ex. environment variable)

- pull model
  - service pulls configuration properties from server
  - how: git, DB, config servers(ex. Spring Cloud Config Server, Hashcorp Valut, ...)
  - benefit
    - centralized configuration: easy to manage, no duplicate
    - transparent decryption of sensitive data
    - dynamic reconfiguration

### 11.3 observable services

- health check API

  - tell if service is available
  - response
    - empty response with appropriate status code
    - or with detail description <- may contain sensitive information
  - implement
    - determine the health of the service instance
  - invoke
    - service registry
    - Docker, k8s has other way(see chapter 12)

- log aggregation

  - aggregate logs that are scattered
  - how service generate log
    - logging library
      - java: Logback, log4j, JUL, SLF4J
    - where to write
      - well-known local filesystem
    - log aggregation infrastructure
      - ELK
        - elastic search: text search-oriented nosql DB
        - logstash: log pipeline that aggregates service log and writes them to elastic search
        - kibana: visualization tool

- distributed tracing
  - assign each external request a unique ID and record how it flows
  - key concepts
    - trace: external request. consists of one or more spans
    - span: represents an operation. key attribute= operation name, start timestamp, end time. can have child spans(=nested operation)
  - components
    - instrumentation library
      - manage the trace and span, add tracing information.(B3 standard)
      - interceptor or AOP
    - distributed tracing server
      - stitch the span together to form complete trace and store in database
- exception tracking
  - problem with tracking exception with logging
    - log files are oriented around single-line, whereas exceptions consist of multiple lines
    - no automatic mechanism to handle duplicate exceptions
  - solution: use exception tracking service. probably that provides client library
- application metrics
  - service report metrics to central server that provides aggregation, visualization, and alerting
  - metrics: (name, value, timestamp, \[dimension\])
    - dimension: name-value pair attributes
  - service should
    - collect its metric
    - expose service metrics: push model(AWS Cloudwatch) / pull model(prometheus)
- audit logging
  - log (identity of the user, action, business object)
  - usually store in database
  - implementations
    - add to business logic
      - straightforward
      - audit logging is intertwined with business logic code -> error prone
    - AOP
      - more reliable
      - only have access to method name and arguments
      - may be challenging to get business object
    - event sourcing
      - automatically provide auditing
      - doesn't record query
