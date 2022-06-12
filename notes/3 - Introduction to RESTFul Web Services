# Introduction
REST is probably the most common way for microservices to communicate but REST does not indicate Microservices.
# HTTP
**HTTP**: Backbone of how REST APIs work and the entire web.
HTTP 1.1 and 2.0 (faster) are highly compatible

Request Methods:
- GET: A request for a resource (ex: html file, javascript file, image)
- HEAD: GET but only asks for meta info without the body

- POST: Create request Post data to a server (ex: form data)
- PUT: Create/update data at a supplied URI.
- DELETE: Delete a resource

- TRACE: Echo a request to see if it was altered by intermediate servers
- CONNECT: Convert the request to transparent TCP/IP tunnel (HTTPS)
- PATCH: partial modification to a source

Safe Methods (Do not cause changes on the server):
- GET
- HEAD
- OPTIONS
- TRACE

Idempotence Methods (Repeated actions have no further effect on the outcome but not enforced by protocol)
- All safe methods
- PUT
- DELETE


HTTP Status Codes:
- **100** series are informational in nature
- **200** series indicate successful requests
    - **200** Okay
    - **201** Created
    - **204** Accepted
- **300** series are redirections
    - **301** Moved Permanently
- **400** series are client errors
    - **400** Bad Request
    - **401** Not Authorized (Don't have security credentials to access this resource)
    - **404** Not Found
- **500** series are server errors
    - **500** Internal Server Error (
    - **503** Service Unavailable (Overload, bad routing, etc)

# REST (Representational State Transfer)
- Representation - Typically JSON or XML
- State Transfer - Typically via HTTP
RESTful web services are de facto standard for web services


REST Terminology:
- Verbs: HTTP Methods (GET, PUT, POST, DELETE)
    - GET to read object from a resource. Safe.
    - PUT Insert (if not found) or update (if found) data. Not Safe
    - POST Insert new object. Not Safe operation.
    - DELETE Deletes resource. Not safe.
- Messages: Payload of the action (JSON/XML)
- URI: Uniform Resource Identifier. Unique string identifying a resource
- URL: Uniform Resource Locator. URI with network info (http://www.example.com)

- Idempotence: Certain operations can be called multiple times without changing the result after the initial application.
- Stateless: Service does not maintain any client state
- HATEOAS: Hypermedia As The Engine Application State. A REST client should then be able to use server-provided links dynamically to discover all available acions and resources it needs.

# Richardson Maturity Model (RMM): Step towards the glory of REST
Created in 2008 to describe maturity of RESTFul services. No working, formal specification of REST.

RMM Levels:
0. The Swap of POX (Plain Old XML)
    - Uses implementing protocol as a transport protocol
    - Uses one URI and one kind of method
    - Old/Not in use anymore
    - Examples: RPC, SOAP, XML-RPC
1. Resources: Breaks services into distinct URIs
    - Uses multiple URIs to identify specific resources:
        - http://www.example.com/product/1234
        - http://www.example.com/product/5687
2. HTTP Verbs: Introduces verbs to implement actions
    - Most common on practice
    - Most RESTFul API mixed with HTTP Verbs:
        - GET /products/1234 - to return product 1234
        - PUT /products/1234 - to update product 1234
        - DELETE /products/1234 - to delete product 1234
3. Hypermedia Controls (Spring provides HATEOS): Provides discoverability, making the API more self documenting
    - Representation now contains URIs which may be useful to consumers
    - Helps client developers explore the resource
    - No clear standard

# Spring Framework & RESTFul Services
- Spring Framework has 3 libraries for creating RESTFul services
    - Spring MVC (Web apps)
    - Spring WebFlux (Reactive web services)
    - Spring WebFlix.fn (Functional programming model used to define endpoints)
- Spring Framework has 2 libraries for consuming RESTFul services
    - Spring RestTemplate (Primary client for consuming services, highly configurable)
    - Spring WebClient (reactive web client)
