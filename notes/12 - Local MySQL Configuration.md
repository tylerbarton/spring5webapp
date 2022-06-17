# Data Source Configuration
- Tells Java how to connect a JDBC complaint Data Source
  - JDBC URL (DB Server host)
  - User Name
  - Password 

Spring Boot:
- Auto configures embedded data source if not provided in config
  - H2, HSQL, Derby supported
  - If Embedded database is on classpath
- H2 is generally preferred for local development
- H2 supports compatibility modes for major databases (SQL)
- Has DB Console application which can be used to browse database
  - automatically configured if springboot-dev-tools is added
- Permanent Datasource: Must provide data source configuration details. Examples:
  - Locally installed MySQL, Postgres
  - DB running local in docker container
  - Deployed database on a server
  - Managed database (Amazon RDS)

## Datasource Configuration:
- Managed via Spring Boot configuration options
  - application.properties
  - application-<profile-name>.properties (For MySQL locally)
  - environment parameters (overrides .properties)
  - Spring Cloud Config


# Configuring H2 for MySQL
[Cheat Sheet](https://h2database.com/html/cheatSheet.html)

1. In application.properties
``spring.datasource.url=jdbc:h2:mem:testdb;MODE=MYSQL
   spring.h2.console.enabled=true``
2. Access h2 console via browser:
`localhost:8080/h2-console`


# MySQL
MySQL default: `localhost:3306`  

MySQL Clients:
  - MySQL Workbench
  - DBeaver (supports MongoDB)
  - SQuirreL SQL
  - Sequel Pro

## Setup
- Create an account with minimal authority in the database (CRUD authorities only)
- Use CRUD + DDL operations due to hibernate managing schema
- Each microservice will have owner user and database schema
  - Simulates unique DB per service

1. Create database in SQL Workbench
2. In your service, create a new application-<profile_name>.properties
3. Enter database details
```
# Example basic configuration
spring.datasource.username=beer_service
spring.datasource.password=password
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
spring.jpa.database=mysql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```



## Configure Spring Profile for MySQL
Example application-localmysql.properties:
```
# Primary Configuration Details 
spring.datasource.username=beer_service
spring.datasource.password=password
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/beerservice?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
spring.jpa.database=mysql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Additional (Not covered in this section)
spring.datasource.hikari.maximum-pool-size=5

spring.datasource.hikari.data-source-properties.cachePrepStmts=true
spring.datasource.hikari.data-source-properties.prepStmtCacheSize=250
spring.datasource.hikari.data-source-properties.prepStmtCacheSqlLimit=2048
spring.datasource.hikari.data-source-properties.useServerPrepStmts=true
spring.datasource.hikari.data-source-properties.useLocalSessionState=true
spring.datasource.hikari.data-source-properties.rewriteBatchedStatements=true
spring.datasource.hikari.data-source-properties.cacheResultSetMetadata=true
spring.datasource.hikari.data-source-properties.cacheServerConfiguration=true
spring.datasource.hikari.data-source-properties.elideSetAutoCommits=true
spring.datasource.hikari.data-source-properties.maintainTimeStats=false

#disable service discovery
spring.cloud.discovery.enabled=false

# Enable logging for config troubleshooting
#logging.level.org.hibernate.SQL=DEBUG
#logging.level.com.zaxxer.hikari.HikariConfig=DEBUG
#logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

### Errors
1. JMS Hibernate Error:  
`Unable to validate user from /172.17.0.1:38786. Username: null; SSL certificate subject DN: unavailable`

In your main method class:
```java
// Disable JMS auto configuration
@SpringBootApplication(exclude = ArthemisAutoConfiguration.class)
```


# Data Source Connection Pooling

Optimization:
- Statements:
  - Prepared Statements: SQL Statements with placeholders for variables (prevents SQL injection attacks)
- Single datasource connection:
  - Cache prepared statements (server level aswell)
  - Server side prepared statements
  - Statement batching

Why Connection Pooling:
- Application server establishes connection to database server (expensive)
  - Call out to db server to get authenticated
  - db server authenticates credentials
  - establish connection
  - establish a session (allocate memory + resources)
- Spring Boot 2.x uses HikariCP (insane speed)
- Every Database Management System (RDMS) accepts max number of connections - each has a cost
- If multiple instances of your microservice, keep number of pool connections lower
- MySQL defaults to a limit 151 connections (Will depend on your hardware)
- Statement caching is good (but consumes memory on server)
- Disabling autocommit can help improve performance


# Hikari
[Hikari Documentation (good resource)](https://github.com/brettwooldridge/HikariCP)

[MySQL Configuration Guide](https://github.com/brettwooldridge/HikariCP/wiki/MySQL-Configuration)

## Quick Start

Property File (Do this):
```
dataSourceClassName=org.postgresql.ds.PGSimpleDataSource
dataSource.user=test
dataSource.password=test
dataSource.databaseName=mydb
dataSource.portNumber=5432
dataSource.serverName=localhost
```

Beer-Serice Example Property File (application-localmysql.properties)
```
spring.datasource.hikari.data-source-properties.cachePrepStmts=true
spring.datasource.hikari.data-source-properties.prepStmtCacheSize=250
spring.datasource.hikari.data-source-properties.prepStmtCacheSqlLimit=2048
spring.datasource.hikari.data-source-properties.useServerPrepStmts=true
spring.datasource.hikari.data-source-properties.useLocalSessionState=true
spring.datasource.hikari.data-source-properties.rewriteBatchedStatements=true
spring.datasource.hikari.data-source-properties.cacheResultSetMetadata=true
spring.datasource.hikari.data-source-properties.cacheServerConfiguration=true
spring.datasource.hikari.data-source-properties.elideSetAutoCommits=true
spring.datasource.hikari.data-source-properties.maintainTimeStats=false
```

HikariConfig method (Not Springboot accepted):
```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/simpsons");
config.setUsername("bart");
config.setPassword("51mp50n");
config.addDataSourceProperty("cachePrepStmts", "true");
config.addDataSourceProperty("prepStmtCacheSize", "250");
config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");

HikariDataSource ds = new HikariDataSource(config);
```

Debugging / Logging (application-localmysql.properties)
```
# Enable logging for config troubleshooting
logging.level.org.hibernate.SQL=DEBUG
logging.level.com.zaxxer.hikari.HikariConfig=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```


