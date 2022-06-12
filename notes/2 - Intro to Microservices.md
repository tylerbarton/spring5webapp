# The Traditional Monolith Application vs Microservices
Definition:
- One Code Base
- One Build System
- Single Executional Program (WAR or EAR file)

Traits:
- Code is stored together
- Usually one database
- Code release is done as one big version
- Scaling is all or nothing

Benefits:
- Development is easy - everything is in one project
- Deployment is easy - only one thing to deploy
- Testing is simplified - only one application to test

Problems:
- Increased complexity
- Can lead to anti-patterns / code-smells
- Difficult to modify - small changes require full redeployment
- Technology Lock In - monolith is coupled to technology stack

# What are Microservices?
Definition:
- Small targeted services
- Each service has its own repository
- Isolated from other services
- Loosely coupled 
  - interaction is done is a technology-agnostic manner 
  - restful web services: HTTP/JSON

Traits:
- Applications are composed using individual microservices
- each service has its own database
- independent deployment 
- scaling of individual services is possible


Benefits:
- Easy to understand & develop
- Better quality (due to limited scope)
- Scalability
- Reliability (Software bugs are isolated)
- Technology stack flexibility (Any languages or technology stack is possible)

Problems:
- Integration testing can be difficult 
- Deployments are more complex due to many applications
- Operational cost (each service: repo, deployment process, database)
- Additional Hardware resources (different services need additional hardware)

**Amazon's Two Pizza Team**: A microservice should be able to be supported by a team you can feed with two pizzas (~12 people)

# What is the Cloud?
Definition:
- Abstraction of physical underlying hardware
- Virtual servers and services 
- Virtualization is not Cloud Based Architecture
- Large level of redundancy

Microservices:
- Typically, multiple instances of microservices are deployed in a cloud environment
- Important to reliability of the application (If one app is terminated / fails, other takes over)
  - Netflix uses Chaos Monkey (Tool that randomly terminates services to ensure reliability)

Microservice Deployment Tools:
- AWS Beanstalk
- AWS ECS / EKS
- (Google) Kubernetes (Big topic)
- Docker Swarm
- Red Hat OpenShift (Wrapper around Kubernetes)
- Cloud Foundry

# Adopting Microservices
- Applications start as monoliths (very well established already in companies)


Breaking up these applications into microservices is known as **decomposing** (art rather than science)
- Strategies include:
  - By Business Capability (Order service)
  - By Domain Objects (Product Service)
  - By action verbs (Payment Service)
  - By noun (Customer Service)

**Single Responsibility Principle**
- Started about Object-Oriented Programming
- Says a class should just have one purpose, do it well
- SRP can be applied to microservices

# Microservice Architecture and Design
Gateway / Load Balancer:
- Endpoint that is exposed to other services
  - Can be on the internet for public APIs
  - more likely to be internal (internal corporate network)
- Abstracts implementation of services 
- Acts as a proxy for network traffic
- Can also act as a load balancer

Service Instances:
- N number of services running (depends on reliability and load requirements)
- Some tools allow for dynamic scalability based on load
  - Think netflix at night (scale up) vs day (scale down)

Database Tier:
- One database per Microservice (this is a guideline)
- High scalable services often have one transaction database
- Organizations will often have more than one database technology
- Not uncommon to mix SQL and NoSQL databases

Messaging (Message Queue):
- Common to expose an API endpoint via a RESTFul API
  - Dependent microservices are often message based
  - Message follow an event or command pattern
- Messaging allows for decoupling and scalability
- Messaging can be used to design a work flow
  - ex: New order, validate order, charge credit card, allocate inventory, ship order

Downstream Services:
- Often an action on a microservice invokes action on multiple downstream services
  - Amazon invokes over 100 per search (search, sponsors (paid promotion), your history, logged search, etc)
  - Placing order might invoke validator order, pay credit card, allocate inventory, ship order

# [12 Factor Applications](https://12factor.net)
Software is delievered as a service 

1. Codebase (One codebase tracked in revision control (github), many deploys)
2. Dependencies (Explicitly Declare and isolate dependencies) - we will be using Maven
3. Config (Store config in the environment) - Build artifact is an immutable object and the configuration should be separate (database credentials) - we will be using spring cloud config
4. Backing Services (Treat backing servicing as attached resources)
5. Build, release, run (Strictly separate build and run stages)
   - *Build* converts code into an executable bundle known as a build
   - *Release* takes the build and combines it with the deploy's config
   - *Run* runs the app in the execution environment by launching some set of the app's processes against a selected release
We will build things into docker images (immutable) and build that into an environment.
6. Processes (Execute the application as one or more stateless processes)
7. Port Binding (Export services via port binding) - multiple processes, port mapping is very important
8. Concurrency (Scale out via the process model)
9. Disposability (Maximize robustness with fast startup and graceful shutdown)
10. Dev/prod Parity (Keep development, staging, and production as similar as possible)
11. Logs (Treat logs as event streams) - create a view across multiple microservices
12. Admin Processes (Run admin/management tasks as one-off processes)
    - Running database migration
    - running a console to run arbitrary code or inspect app's models against the live database