# Spring RestTemplate
Client tool used to interact with Restful Web servers.

**RestTemplate** is the central class within the Spring framework for executing synchronous HTTP requests on the client side.

Mavin Config:
- JUnit 5 (maven-surefire-plugin)
- maven-enforcer-plugin
- spring-snapshots
- spring-milestones

External HostName:
- Don't want hard-coded host name (configured in application.properties in the client)

Methods:
- GET: restTemplate.getForObject(path, object_type)
- POST: restTemplate.postForLocation(path w/o uuid, data)
- PUT: restTemplate.

# HTTP Clients
Definitions:
- **HTTP** - Application Layer: How the client is communicating with the server
- **TCP (Transmission Control Protocol)** - Transport layer:
  - Data is moved in packets between client & server
  - Server listens on a port (ie 80, 443), an ephemeral port is used on client to communicate back
  - Data is divided into packets, transmitted, then **re-assembled**
- **IP (Internet Protocol)** - Internet layer
  - Specification of how packets are moved between hosts (just one packet)

Java I/O:
- Network communication via java.io packages
- low level, lightweight libraries
- TCP/IP connections are made via sockets
- Threads are more costly than a socket connection
  - Modern system can support roughly 100's of thousands of sockets
  - roughly only ~10,000 threads

Blocking & I/O:
- non-blocking IO (NIO): Sets of sockets can be used by a thread
- NIO.2 with async I/O: networking tasks done completely in the background by the OS 

Performance:
- Not uncommon for microservices to have many client connections
- Non-blocking benchmark higher than blocking
- Connection pooling can be used to avoid cost of thread creation & establish connections

Blocking Clients:
- JDK - Java's implementation
- Apache HTTP Client
- Jersey
- OkHttp - may be changing (might be non-blocking)

Non-Blocking Clients:
- Apache Async Client
- Jersey Async HTTP Client
- Netty - Used by Reactive Spring (Video Games)

HTTP/2:
- Better performance than HTTP 1.1
- Uses TCP layer much more efficiently
  - Faster encryption
  - Multiplex streams
  - Binary protocols / compression
  - Reduced latency
- 1.1 and 2 are functionally the same to REST developers
  
  
    HTTP/2 Clients
- Java 9+
- Jetty
- Netty (Has great logging support)
- OkHttp
- Vert.x
- Firefly
- Apache 5.x


# Apache HTTP Client SpringBoot Configuration
Apache Logging:
- In application.properties set `logging.level.org.apache.http=debug`
- Requires Apache http client configuration. Do not set in production version.
- \>\>: request to api
- \<\<: back from api

# Externalize Properties
Spring Skills Assignment:
- Hard coding configuration values is generally considered a bad practice
- HTTP Client properties are currently hard-coded
- Use Spring to externalize the properties

```java
    // Externalized Connection properties
    private final Integer maxTotalConnections;
    private final Integer defaultMaxTotalConnections;
    private final Integer connectionRequestTimeout;
    private final Integer socketTimeout;


    /**
     * Externalized properties constructor
     * Properties found in application.properties
     * @param maxTotalConnections
     * @param defaultMaxTotalConnections
     * @param connectionRequestTimeout
     * @param socketTimeout
     */
    public BlockingRestTemplateCustomizer(@Value("${sfg.maxtotalconnections}") Integer maxTotalConnections,
                                          @Value("${sfg.defaultmaxtotalconnections}") Integer defaultMaxTotalConnections,
                                          @Value("${sfg.connectionrequesttimeout}") Integer connectionRequestTimeout,
                                          @Value("${sfg.sockettimeout}") Integer socketTimeout) {
        this.maxTotalConnections = maxTotalConnections;
        this.defaultMaxTotalConnections = defaultMaxTotalConnections;
        this.connectionRequestTimeout = connectionRequestTimeout;
        this.socketTimeout = socketTimeout;
    }
```


# JPA Entities (Domain)
Represent data that can be persisted to the database. An entity represents a table stored in a database.

Error:
```text
Description:
Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class
```

Resolved by:

applications.properties
```text
spring.sql.init.mode=embedded
spring.cache.jcache.config=classpath:ehcache.xml
spring.datasource.url=jdbc:h2:mem:testdb;MODE=MYSQL
spring.h2.console.enabled=true
```

pom.xml
```xml
<dependency>
 <groupId>com.h2database</groupId>
 <artifactId>h2</artifactId>
</dependency>
```

# Repository
Holds information 