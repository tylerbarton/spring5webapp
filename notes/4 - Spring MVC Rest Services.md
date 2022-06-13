# SFG Beer Works
A set of Spring Applications to showcase microservices and Spring Cloud.

Layout:
- **Beer Consumer**: Creates demand for beer
  - Kotlin
  - WebClient
- **Pub**: Provides beer to **Beer Consumer**, reorders beer from distributor
  - WebFlux.fn
  - Spring Data Mongo
  - MongoDb
- **Distributor**: Restocks beer to pubs, orders beer from Brewery
  - WebFlux
  - Spring Data Mongo
  - MongoDb
- **Brewery**: Brews beer, supplies beer to distributor. Most traditional application
  - Spring MVC
  - Spring Data JPA (Java Persistence API)
  - Hibernate

Enterprise Release:
- Expect fully featured Spring applications
- Applications will have features not covered in the course
  - scheduled tasks 
  - logging 
  - mappers
- Will be working with 'release' artifacts for deployments under Spring Cloud
  - jar files in Maven Central
  - Docker images in Docker hub

# REST Method for GET
Goals:
- Create New Project
- Data Model
- MVC Controller
- Service

0. Create a repository on GIT
1. Create a Spring Project using [Spring Initializer](https://start.spring.io/)
   - Spring Web 
   - Lombok
   - Spring Boot DevTools
2. Setup MVC
3. Using Postman send "http://localhost:8080/api/v1/beer/eb95415b-f8fa-4e60-8bed-45cc4cee49ca" or another UUID using a [generator](https://www.uuidgenerator.net/)

Lombok: 
  - IntelliJ: Preferences | Build, Execution, Deployment | Compiler | Annotation Processors | *Enable annotation processing*
  - Hooks at compile time

Issue:  
Q: Could not autowire / Consider defining a bean of type...  
A: Annotate your ServiceImpl using @Service


Q: Server port 8080 is taken up by another process
A: in resources | application.properties | "server.port=8081"

# GET Endpoint with MVC 
**Goals**:
- Create new DTO for Customer (UUID, string name)
- Use Lombok for DTO
- Create Controller and get by ID URL
- Create Service and service implementation
- Test in postman

# Springboot Developer Tools
- Added to project via artifact 'spring-boot-devtools'
- Automatically disabled when running a packed app (ie java -jar)

Capabilities:
- Automatic Restart (Only restarts your project not the dependencies)
- Templates are cached for performance
- LiveReload automatically triggers browser refresh when resources are changed (**livereload.com for plugin**)


# API Versioning
Important best practice when creating APIs.
ex: "/api/v1/beer"
Typical:
- v1: first release
- v2: second release, notify consumers v1 version is deprecated
- v3: remote v1 (optional), notify consumers v 2 is deprecated

[Semantic Versioning](semver.org):  
Version = MAJOR.MINOR.PATCH  
Common to create a version package  

- **MAJOR** - version for major incompatible API changes - aka breaking changes
  - New required parameter
  - Removal of existing parameter
  - Removal of response value
  - Parameter name change or type
  - Deprecation of a service
- **MINOR** - new functionality - backwards compatible changes
- **PATCH** - backwards compatible bug fixes
  - New optional parameter
  - new response fields
  - new service (endpoint)
  - Bug fixes - behavior change, NOT change to API itself

## GIT API Versioning aka [Semflow](https://github.com/lyndseypadget/semflow)
A GIT version strategy example (cars)
![image](https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/v_table.png)

![git](https://raw.githubusercontent.com/lyndseypadget/semflow/master/images/gitflow_hack.png)

# Issues
### 1. Test Cases: @MockBean is null
```text
java.lang.NullPointerException: Cannot invoke "com.tylerbarton.msscbrewery.services.BeerService.getBeerById(java.util.UUID)" because "this.beerService" is null
```
> tests should match src directory ex:
> com.tyler.barton.msscbrewery.web.controller package must match exactly.