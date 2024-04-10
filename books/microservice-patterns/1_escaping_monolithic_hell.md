# 1. Escaping monolithic hell

## benefits of the monolithic architecture

- simple to develop
- easy to make radical changes to the application
- straightforward to test, deploy
- easy to scale: run multiple instances behind load balancer

## problems of the monolithic architecture

- got too complex
- development is slow
- scaling is difficult
- delivering a reliable monolith is challenging

## scale cube

- x-axis
  - multiple instances
- z-axis
  - spread requests based on an attribute of the request
  - each instance is responsible for only a subset of the data
- y-axis
  - functional decomposition of an application

## microservices

- form of modularity
- each service owns its database
- the services are loosely coupled and communicate only via APIs.

## benefits of the microservice architecture

- enables continuous delivery and deployment of large, complex application

  - why
    - test gets small -> easier and faster to test
    - small unit of deployment
  - business benefits
    - reduces time to market
    - reliable service
    - higher employee satisfaction

- services are small and easily maintained
- services are independently deployable
- services are independently scalable
- enables teams to be autonomous
- introducing new technology is easy
- better fault isolation

## drawbacks of the microservice architecture

- finding the right set of services is challenging
- distributed systems are complex
  -> development, testing, deployment is harder
- deploying features needs careful coordination
- deciding when to adopt the microservice architecture is difficult
