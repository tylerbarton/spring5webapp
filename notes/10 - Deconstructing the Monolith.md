# Deconstructing the Monolith
Breaks down a monolith application into microservice architecture

## Example Project Review (Brewery)
Beer Works - models a brewery
- Beer
- Beer Inventory
- Beer Orders
- Customers

Tasting Room Flow:
- Creates 'demand': brewery needs to consume beer inventory for tasting room orders
- Demand triggers inventory reduction
- Inventory reduction will trigger brewing of beer
- Brewing will create inventory

Technology Stack:
- Spring Boot / Spring Framework
- Hibernate / Spring Data JPA
- Spring MVC
- Project Lombok / Mapstruct
- Thymeleaf for UI Views
  - Lives in `resources/templates`
- Spring Scheduled Tasks (Random Beer Order Task)
- Spring Events (Common theme)

## Tasks
Events using Spring Event Infrastructure that is contained to a single JVM

## Layer Explanations (Domain-Driven Design (DDD) Microservice)
- **Application (Web API)**:
- **Domain model layer**:
  - POJO entity classes
  - Domain entities with data + behavior
  - DDD patterns:
    - Domain entity, aggregate
    - Aggregate root, value object
    - Repository contracts/interfaces
- **Infrastructure Layer / Service**:
  - Data persistence infrastructure
    - repository implementation
  - Data access API
  - Other infrastructure implementation used from the application layer
    - logging
    - cryptography
    - search engine
    - etc.
  - Handles the business logic


# Deconstruction Strategies
## Domain Driven Design (DDD)
definition:
- seen as extension to Object-Oriented Programming
- models complex systems

- **Domain**: Sphere of knowledge, influence, or activity
- **Model**: Asbtractions that describes the selected aspects of a domain used to solve problems related to that domain
- **Ubiquitous Language**: Language around the domain model used by all team members to connect all activities
- **Context**: Statements about model can only be interpreted by context
  - Subsystem
- **Bounded Context**: Subsystem in which a particular model is defined and applicable
  - Helps contain complexity
  - Helps organization

Building Blocks:  
Helps you define implementation details
- **Entities**: Represent identity. Runs through time and across different distinctions (ie: user within a system)
- **Value Objects** immutable objects that describe or compute some characteristic of an object that does not have an identity
- **Domain Events**: An object used to record a discrete event related to a model activity
- **Services**: Handles events. Sometimes, it isn't a thing. Some concepts are not natural to model as objects.
- **Aggregates**: A cluster of entities and value objects.
  - An entity with data and behaviors
  - A data is not an aggregate
- **Repositories**: Query access to aggregate express in the ubiquitous language (specialized service)
  - Only concern is persistence of aggregates
- **Factories**: Like OOP, factories create aggregates.

Example: (Warehouse Management System)  
- Microservices in 'bounded context'
  - Inventory: What quantities are contained in the warehouse
  - Order allocation: 
    - Inbound orders 
    - reserving inventory for those orders
  - Manifest (shipping interface with parcel carriers)
    - Manages shipping
    - Directs people how to pick a product from the warehouse
    - Pack product in boxes / shipping labels
  - Labor Tracking
    - Tracking what service employees do (receiving, orders, etc)
  - Returns

## Example Plan
Each has its own:
- Database / Persistence 
- Models
1. BeerInventoryService: (Manages inv, takes in inv from brewing, provides inventory for orders)
- BEER_INVENTORY

2. BeerService: (Manages products, brewing/manufacturing, list beers + validate order lines for valid beers)
- BEER

3. BeerOrderService: (Manages beer orders from placement to shipment, manage customers (Could be its own microservice), implement call backs to customers)
- BEER_ORDER_LINE
- BEER_ORDER
- CUSTOMER

Projects used:
- mssc-beer-service 
- mssc-beer-order-service
- mssc-beer-inventory-service


# Port Mappings / Saving Default Ports for Services
In resources/application.properties:
`server.port=8080` 

_Possible_ to use 8080 and use virtual networking (Docker).

# SQL Data Initialization
Located:
`resources/data.sql`

application.properties:
`spring.datasource.initialization-mode=EMBEDDED`

Also:

`spring.sql.init.mode=embedded  
spring.cache.jcache.config=classpath:ehcache.xml  
spring.datasource.url=jdbc:h2:mem:testdb;MODE=MYSQL  
spring.h2.console.enabled=true`  


# Adding Caching (using **ehcache**)
Reduce calls to the database. Only calls cache when we are not getting the inventory

1. Set up Cache Config:
`resources/ehcache.xml`

Example configuration:
```xml
<config
        xmlns:jsr107='http://www.ehcache.org/v3/jsr107'
        xmlns='http://www.ehcache.org/v3'>
    <service>
        <jsr107:defaults enable-management="true" enable-statistics="true"/>
    </service>
    
<!--  Here we configure caches-->
    <cache alias="beerCache" uses-template="config-cache"/>
    <cache alias="beerUpcCache" uses-template="config-cache"/>
    <cache alias="beerListCache" uses-template="config-cache"/>

    <cache-template name="config-cache">
        <expiry>
            <ttl unit="minutes">5</ttl>
        </expiry>
        <resources>
            <heap>1</heap>
            <offheap unit="MB">1</offheap>
        </resources>
    </cache-template>
</config>
```

2. Set in application.properties:
`spring.cache.jcache.config=classpath:ehcache.xml`

3. in cache object classes, include `implements Serializable`

4. in java/{com}/config/CacheConfig.java
```java
@EnableCaching
@Configuration
public class CacheConfig {/*Empty*/}
```

5. Set up Caching in Service Implementations api functions (ex: `listBeer()`):
```java
// Can add key="#beerId" else Spring generates key dynamically
@Cacheable(cacheNames="beerCache", condition="#showInventoryOnHand==false")
@Override
// Method would go here
```
```java
// Rest of method
// Set in url
  if (showInventoryOnHand){
            beerPagedList = new BeerPagedList(beerPage
                    .getContent()
                    .stream()
                    .map(beerMapper::beerToBeerDtoWithInventory)
                    .collect(Collectors.toList()),
                    PageRequest
                            .of(beerPage.getPageable().getPageNumber(),
                                    beerPage.getPageable().getPageSize()),
                    beerPage.getTotalElements());
        } else {
            beerPagedList = new BeerPagedList(beerPage
                    .getContent()
                    .stream()
                    .map(beerMapper::beerToBeerDto)
                    .collect(Collectors.toList()),
                    PageRequest
                            .of(beerPage.getPageable().getPageNumber(),
                                    beerPage.getPageable().getPageSize()),
                    beerPage.getTotalElements());
        }
  // Rest of method (such as return...)
```

6. When calling this, you could send `http://localhost:8080/api/v1/beer/{beerId}?showInventoryOnHand=true` to get a live / non-cached update.

