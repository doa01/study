# 12. Deploying Microservices

---

### self-study
- runtime
  - sub-system that is created when the program is running
  - name comes from compile time and runtime

- comparison between concepts similar to runtime system
Type | Description | Examples
--- | --- | ---
Runtime environment | Software platform that provides an environment for executing code | Node.js, .NET Framework
Engine | Component of a runtime environment that executes code by compiling or interpreting it | JavaScript engine in web browsers, Java Virtual Machine
Interpreter | Type of engine that reads and executes code line by line, without compiling the entire program beforehand | CPython interpreter, Ruby MRI, JavaScript (in some cases)
JIT interpreter | Type of interpreter that dynamically compiles code into machine instructions at runtime, optimizing the code for faster execution | V8, PyPy interpreter

---

- vm
  - hypervisor: virtualize physical machine
    - type1: bare metal
      - run on the hardware
    - type2:
      - runs on top of the os
    - virtual hardware
      - emulated hardware
      - hypervisor is responsible for managing and allocating these resrouces to vms
    - guest os: os in vm
- docker
  - virtualize os
  - cgroup, namespace: management

- vmi vs snapshot
https://forum.huawei.com/enterprise/en/snapshots-vs-images-which-one-to-choose/thread/667248320629850112-667213860102352896

- packer
> A machine image is a single static unit that contains a pre-configured operating system and installed software which is used to quickly create new running machines. Machine image formats change for each platform. Some examples include AMIs for EC2, VMDK/VMX files for VMware, OVF exports for VirtualBox, etc.


https://developer.hashicorp.com/packer/docs/intro

---

## Deployment
- deployment: software to production
  - process: what should be performed
  - architecture: structure of the environment that software runs
- history
  - process
    - developer -> operator: separate
    - mid 2000s
      - apache tomcat and jetty. multiple applications on each container
      - PM -> VM
    - now
      - devOps: developer deploys
  - architecture
    - PM -> VM -> container, serverless

- production environment should have
  - Service management interface
    - Enables developers to create, update, and configure services.
    - ideally, REST API invoked by cli, gui
  - Runtime service management
    - must restart crashing service
  - Monitoring
    - provide developers what the service is doing, alert on problem
  - Request routing
    - route requests from users to the service

- 4 deployment options
  - language specific
  - virtual machine
  - container: k8s
  - serverless: lambda



## language-specific packaging format patter
### what & steps
- what is deployed and managed by runtime is language-specific package
  - java: executable jar file, war
  - nodejs: directory of source code and modules
  - golang: operating system-specific executable
- steps
  - 1\. deploy package to the machine
  - 2\. execute the file

### required
- install the necessary runtime(ex. JVM)

### service
  - = one process or group of processes
  - multiple service instances in a single process.
    - ex. multiple java services in single apache tomcat
    - traditional, expensive and heavyweight application server

### benefits

- fast deployment
  - copy runnable & run
  - deploy executable file -> small size of bytes delivered over network
- efficient resource utilization.
  - multiple instances on the same machine or within the same process
  - multiple service instances share the machine and os. \
  more efficient if share the same process

### drawbacks
- lack of encapsulation of the tech stack
  - deployer must known the language specific details
- can not constrain resources per service instance
  - one process can consume all
- lack of isolation between services on the same machine
  - one bad service can affect all others
- determining where to put service is hard
  - deployer should figure out where to deploy

## virtual machine pattern
- deploy by vmi(virtual machine image)
  - vmi: virtual machine setup + service code
  - animator, packer
- benefit
  - encapsulation of tech stack
    - no need to know what application needs
    - service deployment api = vm deployment api
  - isolated service instance
    - each vm has a fixed amount of cpu and memory -> can not steal
  - use mature cloud infrastructure
    - can utilize other functions on cloud
- drawback
  - less efficient resource utilization
    - each application is vm -> less efficient for lightweight app like nodejs, golang
  - relative slow deployments
    - larger size of data over network
    - vm boot up time, ..
  - responsible for patching the os and runtime

## container pattern
- operating system level virtualization
- in the perspective of a process,
  - as if it's on its own machine
  - have its own ip -> no port conflict
- most famous: docker
- can specify cpu, memory, ..

- example with Docker
  - container image
    - filesystem image: application + software needed(jdk,, ...)
  - Dockerfile
    - recipe of container image

      ```Dockerfile
      FROM openjdk:8u171-jre-alpine # base image
      RUN apk --no-cache add curl # install sub software
      CMD java ${JAVA_OPTS} -jar ftgo-restaurant-service.jar # run application when container starts
      HEALTHCHECK --start-period=30s -- interval=5s CMD curl http://localhost:8080/actuator/health || exit 1
      COPY build/libs/ftgo-restaurant-service.jar . # copy application to image
      ```
  - Docker image
    - build
      ```bash
      cd ftgo-restaurant-service
      ../gradlew assemble
      docker build -t ftgo-restaurant-service
      ```
    - push to registry
      - registry: cloud storage/marketplace of docker image
      ```bash
      docker tag ftgo-restaurant-service registry.acme.com/ftgo-restaurant-service:1.0.0.RELEASE
      ```
    - layered file system: only changed part are transferred -> efficient

  - running a docker container
    - pull image from the registry & start
    ```bash
    docker run \
      -d  \ # as background
    --name ftgo-restaurant-service  \ # name of the container
    -p 8082:8080  \ # port binding with the host
    -e SPRING_DATASOURCE_URL=... -e SPRING_DATASOURCE_USERNAME=...  \ # environment variables
    -e SPRING_DATASOURCE_PASSWORD=... \
    registry.acme.com/ftgo-restaurant-service:1.0.0.RELEASE # image to run
    ```
  - docker has problems
    - no restart on container crash
    - hard to manage all dependent services like DB, ... \
      => `Docker Compose` : start and stop containers as a group. \
      but still... in one machine

- benefits
  - fast to build
  - Encapsulation of the technology stack. the API for managing your services becomes the container API
  - Service instances are isolated.
  - Service instances’s resources are constrained.

- drawbacks
  - need to manage container image
  - must manage container infrastructure and (possibly) vm infrastructure

## kubernetes
- Kubernetes is a ~Docker orchestration framework~
  - Resource management
    - Treats a cluster of machines as a pool of CPU, memory, and storage volumes, turning the collection of machines into a single machine.
  - Scheduling
    — Selects the machine to run your container considering resource requirements of the container and each node’s available resources
  - Service management
    - has the concept of name and versioned services
    - guarantee the desired number of healthy instances
    - load balancing
    - rolling update and rollback
- docker swarm lost

### k8s architecture
- unit of k8s: cluster
- glossary
  - master: manage cluster. small number
  - node: worker that runs pods. large number
  - pod: unit of deployment. a set of containers
- master components
  - k8s api-server: rest api server
  - etcd: key-value noSQL database. store cluster data
  - scheduler: select node to run a pod
  - controller manager: run the controller
- node components
  - kublet: create and manage the pod
  - kube-proxy: manage network including load balancing
  - pod: application services

- key concepts(resource)
  - pod
    - basic unit of deployment
    - one or more containers
    - share ip address and storage volume
    - ephemeral: if pod is dead, eveything is gone
  - deployment
    - controller that ensures the state of the pod
  - service
    - static/stable network location definition
    - can call the application with dns, ip of the service
  - configmap
    - collection of name-value pairs
  - secret
    - like configmap but stores credentials

- loadbalancer
  - can use AWS ELB, ...
  - k8s's loadbalancer lacks configuration

- zero downtime deployment
  - k8s supports rolling upgrade
    (have other options too)
  - you do
    - build & push image
    - edit k8s yaml & apply
  - can rollback
  - but,... no canary deployment...

### service mesh
- staging is needed. or deploy & test then expose to end users
- service mesh provides rule-based load balancing and traffic routing that lets you safely run multiple versions of your services simultaneously

- traffic management
- security
- telemetry
- policy enforcement : quota & ratelimit

## TODO
- kubectl
- how it works
