Section is dedicated to annotation processing

# Project Lombok
## Overview:
- Hooks into annoation processor API which generates code at Compile Time
- Annotation Processing in Compile Settings must be enabled:
> Preferences | Build, Execution, Deployment | Compiler | Annotation Processors | Enable annotation processing

## Features:
- **@NonNull**: Method parameters not null; Throws NullPointerException
- **@Cleanup**: Cleanup in finally property
- **@Getter**: getter methods for all properties. 
  - `lazy=true` will calculate value first time and then caches it to speed up
- **@Setter**: setter methods for all properties
- **@EqualsAndHashCode**
  - Generates impl of equals(Object other) and hashCode()
  - Default: use non-static and notransient properties
  - Optional: exclude specific properties
- **@NoArgsConstructor**: Generates a no args constructor but will error if final fields (can force)
- **@RequiredArgsConstructor**: Generates a constructor for all fields that are final or marked @NonNull
- **@AllArgsConstructor**: Generates a constructor for all properties
- **@Data**: Combines:
  - @Getter
  - @Setter
  - @ToString
  - @EqualsAndHashCode
  - @RequiredArgsConstructor
- **@Value**: immutable variant of @Data; All fields are made private and final by default
- **@Builder**: implements the 'builder' pattern for object creation. Removes a lot of work.
- **@SneakyThrows**: Throws checked exceptions without declaring in calling method's throws clause
- **@Synchronized**: safer implementation of Java's synchronized
- **@Slf4j**: Incorporates the generic logging framework 

Variable types:  
**val**: Figures out the type. Usually marked final for immutable 

## Configuration:
In Intellij:
> Preferences | Build, Execution, Deployment | Compiler | Annotation Processors | Enable annotation processing

Import into pom.xml:
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```


Error:
### 1. lombok.javac.apt.LombokProcessor cannot access class com.sun.tools.javac
Switch to at least version 1.18.22

# MapStruct
## Overview:
- MapStruct is a code generator for mapping between Java bean types
  - ie: BeerDto -> Beer(Entity)
- Generates source code (unlike Lombok)
- Works by programmer creating **interface**, MapStruct generates implementation
- Complex mappings can be done via _abstract classes_ 

## Features:
- Properties of the same name / same type automatically mapped
- Different property names are configured via annotation properties
- Different types can be referenced with additional mapper implementations
- Mappers can reference other mappers
  - ie: Order Mapper can use the Order Line Mapper
- Can be configured to annotate generated mappers as Spring Components


## Configuration:

```xml
<properties>
  <java.version>11</java.version>
  <!--		Define package versions here-->
  <mapstruct.version>1.4.2.Final</mapstruct.version>
  <org.lombok.version>1.18.22</org.lombok.version>
</properties>
```

Works with Lombok but requires `lombok-mapstruct-binding`
```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>${mapstruct.version}</version>
</dependency>
```

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
        <version>${mapstruct.version}</version>
      </path>
      <path>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${org.lombok.version}</version>
      </path>
      <path>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok-mapstruct-binding</artifactId>
        <version>0.2.0</version>
      </path>
    </annotationProcessorPaths>
    <compilerArgs>
      <compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>
    </compilerArgs>
  </configuration>
</plugin>
```


## Implementation:
Basic:
```java
/**
 * Uses MapStruct to convert BeerDto <==> Beer (Entity)
 * Assumes BeerDto and Beer have the exact same properties
 */
@Mapper
public interface BeerMapper {
    BeerDto beerToBeerDto(Beer beer);
    Beer beerDtoToBeer(BeerDto dto);
}
```


# Why We Use A Mapper
**Entity**: Data that is serializable into a database  
**DTO (Data Transfer Object)**: Data that is sent to an API

We use a mapper to convert between the two types of objects.

### DateTime Conversion
Entity: Timestamp (java.sqlTimestamp)  
DTO: OffsetDateTime (java.time.OffsetDateTime) 

```@Mapper(uses={DateMapper.class})```

Implementation:  
```java
import org.springframework.stereotype.Component;

import java.sql.Timestamp;
import java.time.OffsetDateTime;
import java.time.ZoneOffset;

/**
 * Converts between Timestamp and OffsetDateTime
 */
@Component // Required to be picked up
public class DateMapper {
    /**
     * Converts to OffsetDatetime
     * @param ts Timestamp from an enetity
     * @return OffsetDateTime version of the input Timestamp
     */
    public OffsetDateTime asOffsetDateTime(Timestamp ts){
        if (ts != null){
            return OffsetDateTime.of(ts.toLocalDateTime().getYear(), ts.toLocalDateTime().getMonthValue(),
                    ts.toLocalDateTime().getDayOfMonth(), ts.toLocalDateTime().getHour(), ts.toLocalDateTime().getMinute(),
                    ts.toLocalDateTime().getSecond(), ts.toLocalDateTime().getNano(), ZoneOffset.UTC);
        } else {
            return null;
        }
    }

    /**
     * Converts to a Timestamp
     * @param offsetDateTime OffsetDateTime to convert
     * @return Timestamp version of input offsetDateTime
     */
    public Timestamp asTimestamp(OffsetDateTime offsetDateTime){
        if(offsetDateTime != null) {
            return Timestamp.valueOf(offsetDateTime.atZoneSameInstant(ZoneOffset.UTC).toLocalDateTime());
        } else {
            return null;
        }
    }
}
```


# [CircleCi](http://circleci.com)
After committing, gives a notification if the build is broken.
