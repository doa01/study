# 10. Testing microservices: part 2

## 10.1. writing integration test

### how - strategies

- 1\. test each service's adapter, and its supporting classes
  - simple and faster
- 2\. use contract
  - consumer-side test
    - test consumer's adapter
    - use stub to simulate provider
  - provider-side test
    - test provider's adapter
    - use mock for the adapter's dependencies

### persistence integration test

- verify that a service's database access logic works as expected
- integration test don't make assumption about the state of the database
- use Docker for database

- phase
  - setup: setup the database by creating schema and initialize to some state
  - execute: perform database operation
  - verify: make assertion
  - teardown: optional. undo some changes

### REST-based request/response style

- consumer driven contract test
  - verify that API meets client's expectation
  - consists of http request and reply
  - test API Gateway and Domain Service
  - consumer-side API Gateway integration test
    - verify that API meets client's expectation
    - mock order service
    - test proxy
      - verify expected status for request
  - provider-side test
    - test class is generated
    - test controller

### publish/subscribe style

- domain event instead of http request/response
- consumer-side test
  - verify handler invokes its dependencies
- provider-side test
  - verify event is published

### asynchronous request/response

- message instead of domain event
- consumer-side test
  - command message proxy class sends correctly structured command message and correctly process reply message
- provider-side test
  - correctly handles a command message and send correct reply

## 10.2 component test

- component test
  - verify the behavior of a service in isolation
  - describe externally visible behavior from the perspective of client, rather than in terms of the internal implementation
- scenario to test
  - given: setup
  - when: execute
  - then: verify
- frameworks
  - Gherkin: DSL for writing executable specifications
  - cucumber: automated testing framework that execute tests written in Gherkin. \
    available in variety of languages
- designing component tests
  - in-process component test
    - run service with in-memory stub and mocks for its dependencies
    - good: simpler to write and faster
    - bad: not testing the deployable service
    - stubbing
      - Spring Cloud Contract. \
        may be heavy weight. need to deploy jar files to maven repository
  - out-of-process component test
    - package service in production-ready format and run in separate process
    - good: improves test coverage, because close to what is deployed
    - bad: complex to write, slower to execute and more brittle than in-process component test. also consider how to stub application services

## 10.3 end to end test

- test entire application
- design
  - user journey test: a single test that does all things that user would do
- run all the application service with Docker Compose
