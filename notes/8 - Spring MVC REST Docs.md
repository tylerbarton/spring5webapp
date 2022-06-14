# Spring REST Docs (Documentation Generator)
[Documentation](https://spring.io/projects/spring-restdocs)
### Overview:
Tool to generate documentation. Important for public-facing APIs.
- Hooks into your tests for your controllers to generate documentation snippets. Supports:
  - Spring MVC Test
  - WebTestClient (WebFlux)
  - REST Assured

- Default Snippets:
  - curl-request
  - http-request
  - http-response
  - httpie-request
  - request-body
  - response-body
- Options:
  - Markdown rather than Asciidoctor
  - Maven and Gradle may be used for the build process
  - You can package the documentation to be served as static content via Spring Boot
  - Third party extension **restdocs-asi-spec** to generate OpenAPI 3 specifications
### Implementation (Maven)
1. add to pom.xml:
```xml
<dependency>
    <groupId>org.springframework.restdocs</groupId>
    <artifactId>spring-restdocs-mockmvc</artifactId>
    <scope>test</scope>
</dependency>
```

2. add to pom.xml plugins:
```xml
<plugin>
    <groupId>org.asciidoctor</groupId>
    <artifactId>asciidoctor-maven-plugin</artifactId>
    <version>1.5.3</version>
    <executions>
        <execution>
            <id>generate-docs</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>process-asciidoc</goal>
            </goals>
            <configuration>
                <backend>html</backend>
                <doctype>book</doctype>
            </configuration>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.springframework.restdocs</groupId>
            <artifactId>spring-restdocs-asciidoctor</artifactId>
            <version>${spring-restdocs.version}</version>
        </dependency>
    </dependencies>
</plugin>
```

3. create directory `src/main/asciidoc`

4. create `src/main/ascii/index.adoc`

Can use this template:
```
= TITLE
AUTHOR;
:doctype: book
:icons: font
:source-highlighter: highlightjs

Sample application demonstrating how to use Spring REST Docs with JUnit 5.

`BeerOrderControllerTest` makes a call to a very simple service and produces three
documentation snippets.

One showing how to make a request using cURL:

include::{snippets}/orders/curl-request.adoc[]

One showing the HTTP request:

include::{snippets}/orders/http-request.adoc[]

And one showing the HTTP response:

include::{snippets}/orders/http-response.adoc[]

Response Body:
include::{snippets}/orders/response-body.adoc[]


Response Fields:
include::{snippets}/orders/response-fields.adoc[]
```

5. In your test code:
```java 
@ExtendWith(RestDocumentationExtension.class)
@AutoConfigureRestDocs          // Automatically configures REST docs
```

### Documenting
Spring REST docs creates the documents when `package` is called in Maven.

#### Path Parameters:
Important:
```
REPLACE: import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
WITH: import static org.springframework.restdocs.mockmvc.RestDocumentationRequestBuilders.*;
```

example:
```java
void getBeerById() throws Exception {
    mockMvc.perform(get("/api/v1/beer/{beerId}", UUID.randomUUID().toString())
                    .accept(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())
            // REST docs - path parameters
            .andDo(document("v1/beer", pathParameters(
                    parameterWithName("beerId").description("UUID of desired beer to get.")
            )));
}
```

#### Query Parameters
URL-encoded parameters

example:
```java
@Test
void getBeerById() throws Exception {
    mockMvc.perform(get("/api/v1/beer/{beerId}", UUID.randomUUID().toString())
                    .param("iscold", "yes") // Query Param
                    .accept(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())

            .andDo(document("v1/beer", 
                    pathParameters( // Path Param
                        parameterWithName("beerId").description("UUID of desired beer to get.")
                    ),
                    requestParameters( // Query Param
                            parameterWithName("iscold").description("Is Beer Cold Query param")
                    )
            ));
}
```


#### Response

ex:
```java
@Test
void getBeerById() throws Exception {
    mockMvc.perform(get("/api/v1/beer/{beerId}", UUID.randomUUID().toString())
                    .param("iscold", "yes") // Query Param
                    .accept(MediaType.APPLICATION_JSON))
                    .andExpect(status().isOk())

            .andDo(document("v1/beer",
                    pathParameters( // Path Param
                        parameterWithName("beerId").description("UUID of desired beer to get.")
                    ),
                    requestParameters( // Query Param
                            parameterWithName("iscold").description("Is Beer Cold Query param")
                    ),
                    responseFields( // response Fields
                        fieldWithPath("id").description("Id of Beer"),
                        fieldWithPath("version").description("Version number"),
                        fieldWithPath("createdDate").description("Date Created"),
                        fieldWithPath("lastModifiedDate").description("Date Updated"),
                        fieldWithPath("beerName").description("Beer Name"),
                        fieldWithPath("beerStyle").description("Beer Style"),
                        fieldWithPath("upc").description("UPC of Beer"),
                        fieldWithPath("price").description("Price"),
                        fieldWithPath("quantityOnHand").description("Quantity On hand")
                    )
            ));
}
```

#### Requests
`.andDo(document("api_path", restFields(fieldwithPath(""))));`

#### Validation Constraints


#### Auto Generation
_Note:_ This also gathers type information 

1. Create snippet in `test/resources/org/springframework/restdocs/templates/request-fields.snippet`
```snippet
    |===
        |Path|Type|Description|Constraints

    {{#fields}}
        |{{path}}
        |{{type}}
        |{{description}}
        |{{constraints}}

    {{/fields}}
        |===
```

2. Add to controller test class:
```java
/**
 * Used for documentation of constraints on fields
 */
private static class ConstrainedFields {

    private final ConstraintDescriptions constraintDescriptions;

    ConstrainedFields(Class<?> input) {
        this.constraintDescriptions = new ConstraintDescriptions(input);
    }

    /**
     * Used with request-fields.snippet to generate information on constraints
     * @param path Path of the file
     * @return Description with constraint descriptions.
     * @implNote ConstraintDescriptions fields = new ConstraintDescriptions(BeerDto.class);
     */
    private FieldDescriptor withPath(String path) {
        return fieldWithPath(path).attributes(key("constraints").value(StringUtils
                .collectionToDelimitedString(this.constraintDescriptions
                        .descriptionsForProperty(path), ". ")));
    }
}
```

3. Add to test method
```java
.andDo(document("v1/beer",  // Request documentation
                            requestFields(
                                    fields.withPath("id").ignored(),      // client should not send any of the ignored properties
                                    fields.withPath("version").ignored(),
                                    fields.withPath("createdDate").ignored(),
                                    fields.withPath("lastModifiedDate").ignored(),
                                    fields.withPath("beerName").description("Name of the beer"),
                                    fields.withPath("beerStyle").description("Style of Beer"),
                                    fields.withPath("upc").description("Beer UPC").attributes(),
                                    fields.withPath("price").description("Beer Price"),
                                    fields.withPath("quantityOnHand").ignored()
                            )
                        ));
```


#### Errors
1. > SnippetException 
   
Potentially thrown if all fields are not properly documented - Try looking at your fieldWithPath() calls and see if you missed a parameter.

2. > Unable to make protected native java.lang.Object java.lang.Object.clone() throws java.lang.CloneNotSupportedException accessible: module java.base does not "opens java.lang" to unnamed module @39c87b42
   
Two Options:
  1. Set asciidoctor to 1.5.8
  2. Use following dependency:
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-site-plugin</artifactId>
  <version>3.10.0</version>
  <dependencies>
    <dependency> <!-- Add Asciidoctor Doxia Parser Module -->
      <groupId>org.asciidoctor</groupId>
      <artifactId>asciidoctor-maven-plugin</artifactId>
      <version>2.2.2</version>
    </dependency>
  </dependencies>
</plugin>
```

### Customize URI (@AutoConfigureRestDocs)
If you have a development server and you want the requests to go to that

```java
@AutoConfigureRestDocs(uriHost = "localhost", uriPort = 8080)  
```

Confirm in `target/generated-snippests/v1/beer/curl-request.adoc`

### Generating Final Documentation
1. Run package 
2. Find documents in target/generated-docs/index.html
3. Paste into pom.xml:
```xml
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>prepare-package</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <outputDirectory>
                    ${project.build.outputDirectory}/static/docs
                </outputDirectory>
                <resources>
                    <resource>
                        <directory>
                            ${project.build.directory}/generated-docs
                        </directory>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

# Errors

1. RestDocuemtnationExtension.class not importing (@ExtendWith(RestDocumentationExtension.class))

`import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;`