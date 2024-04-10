# 2. Decomposition strategies

## what is the microservice architectures

- what is microservice architecture exactly?
  -> functional decomposition
- definition of software architecture
  > the software architecture of a computing system is the set of structures needed to reason about system, which comprise software elements, relations among them, and properties of both
- why decomposition is important

  - division of labor and knowledge
  - defines how the software elements interact

- 4+1 view model of software architecture

  - logical view
    - what developers create
    - elements: classes, packages
    - relations: relationships between them
  - implementation view
    - output of build system
    - elements: modules(jar files) and components(war files ar executables)
    - relations: their dependencies
  - process view
    - running components
    - elements: processes
    - relations: inter-process communication
  - deployment view
    - processes running on "machines"
    - elements: machines and processes
    - relations: networking
  - scenario

- why architecture matters
  - two categories of requirements
    - functional requirements: what the application must do
    - quality of service <- architecture helps satisfy this

## overview of architectural styles

- layered architecture style
  - layers
    - presentation layer: implements the user interface or external APIs
    - business layer: business logic
    - persistence layer: logic of interacting with the database
  - drawbacks
    - single presentation layer
      - there can be many systems that invoke an application
    - single persistence layer
      - application can have more than single database
    - defines the business logic layer as depending on the persistence layer
      - prevents testing the business logic without the database
- hexagonal architecture style
  - inbound adapter
    - handles request from outside by invoking business logic
  - outbound adapter
    - invoke external application by business logic
  - port
    - defines set of operations and how the business logic interacts with outside
    - inbound port: API exposed by business logic
    - outbound port: how business logic invokes external applications
  - benefits
    - decoupling business logic from presentation logic, data access logic

## service

- what is a service?
  - standalone, independently deployable software that implements some useful functionality
  - two types of operations
    - command: performs action and updates data
    - query: gets data
- the size of a service s mostly unimportant
  - goal: design a service capable of being developed by a small team with minimal lead time and with minimal collaboration with other teams

## defining an application's microservice architecture

- steps
  - step1. identify system operations
  - user stories and scenarios
    - as a consumer, i want to ...
    - as a restaurant, i want to ...
  - create domain models
  - define system operations
    - commands: parameters, return value and behavior(precondition, post-condition)
    - queries
  - step2. identify services
  - createOrder, acceptOrder, ...
  - define business capabilities = what an organization does
    - mapping business capabilities to services is subjective
  - decompose by subdomain
    - subdomain: part of a domain
    - bounded context: the scope a domain model. a code artifacts that implement the model
  - step3. define service apis ans collaboration
  - OrderService --verifyOrder()--> RestaurantService
- decomposition guideline
  - Single Responsibility Principle
    > A class should have only one reason to change
  - Common Closure Principle
    > The classes in a package should be closed together against the same kinds of changes. A change that effects a package affects all the classes in that package
- obstacles to decomposing an application into services
  - network latency
  - reduced availability due to synchronous communication
  - maintaining data consistency across services
  - obtaining a consistent view of the data
  - god classes preventing decomposition
