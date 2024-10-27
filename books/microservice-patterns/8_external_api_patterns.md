# 8. External API patterns
## External API design issues
- various clients
    - web applications browser-based, with consumers / internal admin ui
    - javascript application
    - mobile app
    - app written by third party developers
- some may inside/outside firewall
    - inside firewall: better network(bandwidth, latency)
- easy architecture: client calls service directly
    - drawbacks
        - client need to make multiple requests -> high latency
        - lack of encapsulation: client should know all the services
        - service may use client unfriendly ipc 

## API gateway pattern
- api gateway: a service that is the entry point into the applications from the outside world
- functions
    - request routing
    - api composition
    - protocol translation: RESTful / gRPC
    - provides client specific api: web/mobile/...
    - edge functions
        - request-processing function implemented at the edge of an application
        - examples
            - authentication
            - authorization
            - rate limiting
            - caching
            - metric collection
            - request logging
        - implemented at
            - backend service
            - edge service -> separation of concern, good with multiple api gateways
            - api gateway -> less network hop(->better latency), less complexity
- architecture
    - api layer: client-specific
    - common layer: shared functions including edge functions
- api gateway ownership model
    - option1: separate teams get responsible for the API gateway
    - option2(better): client team - api layer, api-gateway team - common module

## Backend for Frontend pattern
- each client has its own api gateway
- common functionality: shared library implemented by api gateway team
- benefits
    - clean responsibility
    - isolation between clients -> enhance reliability
    - independently scalable
    - reduce startup time since it becomes smaller, simpler

## benefits and drawbacks of API gateway
- benefits
    - encapsulates internal structure
- drawbacks
    - can be development bottleneck
    - must update gateway to expose their APIs


## design issues of API gateway
- performance, scalability, maintainability
    - use async(nonblocking) i/o
    - use reactive model instead of callback for maintainability
- handle partial failure
    - put multiple instances behind a load balancer
    - use circuit breaker pattern to handle failed request / unacceptably high latency request

## implementing api gateway
- off-the-shelf
    - aws api gateway
    - aws application load balancer
    - kong, traefik

- developing own api gateway
    - netflix zuul
    - spring cloud gateway