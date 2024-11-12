# 13. Refactoring To Microservices

## mindset
- pain points of monolithic architecture
    - slow delivery
    - buggy software release
    - poor scalability

- don't rewrite but refactor
    - rewrite
        - no benefit until done
        - risky
    - refactor
        - increase of benefit over time(techinically, business support)
        - can migrate high value first

- minimize changes to the monolith
    - small changes first
    - only thing required is a deployment pipeline with automated testing


## strategy
- new feature as services
- separate presentation tier and backend
- extract functionality from monolith and make into services


### new feature as services

- additional architecture
    - api gateway(routing)
        - new function -> new service
        - old function -> monolith
    - integration glue code
        - handle dependency between monolith, service
            - access data and invoke functions in monolith
        - form: adapters in monolith and service

- when to make a new service
    - ideally: every new feature
    - but sometimes, too small to be a service. \
        -> first implement in monolith and extract later

- benefit
    - reduce the growth of monolith
    - launched new service's good progress/deployment demonstrates the value of msa

### separate presentation tier and backend
- benefit
    - decoupling
    - remote API that can be used in microservice of the future
- drawback
    - partial solution

### extract functionality from monolith and make into services
- hard and time consuming
#### two main challenges
- splitting domain models
    - problem: class in monolith need to reference the class that has moved to the service
        - solution: reference with primary key like aggregate in DDD
        - caveat: client should get same response like before
            -> to solve this, data replication
    - problem2: responsibility entanglement
        - solution: later
- refactoring database
    - replicate data
- need good selection of what to extract

