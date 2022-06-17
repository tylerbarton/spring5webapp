# Content
- Establish Maven BOM (build materials) to reduce duplication in Maven POMs
- Set up MySQL for database
- Transaction to JMS to publish events as messages
- Build Saga's to coordinate microservice actions for events (ie order allocation)

Projects used:
- mssc-beer-service
- mssc-beer-order-service
- mssc-beer-inventory-service


# Maven Bill of Materials (BOM)
Definition:
- Build materials for services & applications
- BOM is similar to Spring Parent POM

Features:
- Set common Maven Properties
- Set common Maven Plugins and configuration
- Set dependency versions
- Set common dependencies
- Set common build profiles
- Set just about any inheritable property which is common

## Creation
Create a new repository & Maven project that other projects will inherit from

_tip:_ right-click pom.xml > Maven > show effective pom

1. `New Project | Maven`

2. In `pom.xml`

Example starter:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <groupId>guru.springframework</groupId>
    <artifactId>mssc-brewery-bom</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

</project>
```

# Commons
How to configure common properties in a BOM

## Properties:

```xml
<properties>
<!--    ...-->
</properties>
```

## Dependency Management (Versions)
Centralize versions of dependencies used.

```xml
<properties>
<!--    Declare versions here-->
    <org.mapstruct.version>1.3.0.Final</org.mapstruct.version>
</properties>

<dependencyManagement>
<!--    version management to declare a specific version for certain projects-->
    <dependencies>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${org.mapstruct.version}</version>
<!--            Could do scope: <scope>test</scope>-->
        </dependency>
    </dependencies>
</dependencyManagement>
```

## Dependencies
Dependencies shared to all projects

```xml
<dependencies>
    <dependency>
<!--        Include in here-->
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>

```

## Build Plugins & Configurations


Example:
```xml
 <build>
        <plugins>
<!--            Maven Plugin-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            
<!--            Configures auto-clean-->
            <plugin>
                <artifactId>maven-clean-plugin</artifactId>
                <executions>
                    <execution>
                        <id>auto-clean</id>
                        <phase>initialize</phase>
                        <goals>
                            <goal>clean</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>          
        </plugins>
</build>
```

### MapStruct & Lombok
Common to add the configuration for MapStruct & Lombok:

```xml
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                    </annotationProcessorPaths>
                    <compilerArgs>
                        <compilerArg>
                            -Amapstruct.defaultComponentModel=spring
                        </compilerArg>
                    </compilerArgs>
                </configuration>
            </plugin>

```

# Maven Enforcer
Sets up build rules for an organization
```xml
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <executions>
                    <execution>
                        <id>enforce-versions</id>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                        <configuration>
                            <rules>
                                <requireMavenVersion>
                                    <version>[3.6.0,)</version>
                                </requireMavenVersion>
                                <requireJavaVersion>
                                    <version>11</version>
                                </requireJavaVersion>
                                <requireReleaseDeps>
                                    <onlyWhenRelease>true</onlyWhenRelease>
                                    <message>Release builds must not have on snapshot dependencies</message>
                                </requireReleaseDeps>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

- Require Version: `<version>11</version>`  
- Require Version at least: <version>[3.6.0,)</version>
- Require Java Version
```xml
<requireJavaVersion>
    <version>11</version>
</requireJavaVersion>
```


# Completed Example
Completed-Example:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>

    <groupId>guru.springframework</groupId>
    <artifactId>mssc-brewery-bom</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>

    <licenses>
        <license>
            <name>GNU General Public License v3.0</name>
            <url>https://www.gnu.org/licenses/gpl.txt</url>
        </license>
    </licenses>

<!--    Organization -->
    <organization>
        <name>Spring Framework Guru</name>
    </organization>

<!--    Who developed this -->
    <developers>
        <developer>
            <name>John Thompson</name>
            <organization>Spring Framework Guru</organization>
        </developer>
    </developers>

<!--    Common Properties across services -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <org.mapstruct.version>1.3.0.Final</org.mapstruct.version>
        <jaxb.version>2.3.0</jaxb.version>
        <awaitility.version>3.1.6</awaitility.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>javax.xml.bind</groupId>
                <artifactId>jaxb-api</artifactId>
                <version>${jaxb.version}</version>
            </dependency>
            <dependency>
                <groupId>com.sun.xml.bind</groupId>
                <artifactId>jaxb-core</artifactId>
                <version>${jaxb.version}</version>
            </dependency>
            <dependency>
                <groupId>com.sun.xml.bind</groupId>
                <artifactId>jaxb-impl</artifactId>
                <version>${jaxb.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>${org.mapstruct.version}</version>
            </dependency>
            <dependency>
                <groupId>org.awaitility</groupId>
                <artifactId>awaitility</artifactId>
                <version>${awaitility.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${org.mapstruct.version}</version>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-clean-plugin</artifactId>
                <executions>
                    <execution>
                        <id>auto-clean</id>
                        <phase>initialize</phase>
                        <goals>
                            <goal>clean</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${org.mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                    </annotationProcessorPaths>
                    <compilerArgs>
                        <compilerArg>
                            -Amapstruct.defaultComponentModel=spring
                        </compilerArg>
                    </compilerArgs>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <executions>
                    <execution>
                        <id>enforce-versions</id>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                        <configuration>
                            <rules>
                                <requireMavenVersion>
                                    <version>[3.6.0,)</version>
                                </requireMavenVersion>
                                <requireJavaVersion>
                                    <version>11</version>
                                </requireJavaVersion>
                                <requireReleaseDeps>
                                    <onlyWhenRelease>true</onlyWhenRelease>
                                    <message>Release builds must not have on snapshot dependencies</message>
                                </requireReleaseDeps>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```


# Parent POM Configuration

## Local
1. Run Maven `install` on the BOM project
   - Installs this to Maven directory on your machine
2. In your child project pom.xml, change parent:
```xml 
    <parent>
        <groupId>com.github.sfg-beer-works</groupId>
        <artifactId>sfg-brewery-bom</artifactId>
        <version>1.0.17</version>
    </parent>
```
3. Run Maven `verify` on the child project to ensure the project still works
## Released
You can upload packages to Maven Central and use the artifactId in the parent field in pom.xml

[GitHub Organization Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry)

# IntelliJ Tips & Tricks
One IntelliJ workspace/window for all microservices in a directory:

1. `file | new project | empty project`
2. On GitHub, clone your repositories into this new project `git clone ...`
3. `file | new module from existing sources`
4. Select your folder
5. "import Maven project automatically" open