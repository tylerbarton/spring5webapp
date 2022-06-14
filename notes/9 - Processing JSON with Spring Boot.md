# JSON
**JSON**(Javascript Object Notation) is a popular language across all major programming languages.

Definitions:
- **Serialization**: Convert a Java object to a JSON object
- **Deserialization**: Convert JSON object to Java object

JSON Libraries:
- Jackson (Spring Boot Default)
- GSON (Google's Library, potentially faster)
- JSON-B

## HTTP Methods
GET:
- Read Java Object
- Convert to JSON string
- Send to client

POST:
- Spring reads the body
- Deserialize JSON payload into Java object


# Jackson (JSON Library)
Library for parsing JSON in Springboot  
Can be configured in application.properties

Common Annotations:
- **@JsonProperty**: Allows you to set the property name
- **@JsonFormat**: Gives you control over how property is serialized (helpful for Dates)
- **@JsonUnWrapped**: Flattens a property
- **@JsonView**: Allows you to configure virtual 'views' of objects
- **@JsonManagedReference, @JsonBackReference**: Mapping embedded items
- **@JsonIdentityInfo**: Allows you to specify a property to determine object identity
- **@JsonFilter**: Used to specify a programmatic property filter
  - >  @JsonFormat(pattern="yyyy-MM-dd'T'HH:mm:ssZ", shape = JsonFormat.Shape.STRING)  
  @Null  
  private OffsetDateTime createdDate;
  - >  @JsonFormat(shape=JsonFormat.Shape.STRING) 
    @Positive  
    @NotNull  
    private BigDecimal price;  

Serialization Annotations:
- **@JsonAnyGetter**: Takes a Map property and serializes the key to property names, values to values
- **@JsonGetter**: Allows you to identify a method as a 'getter', which will be serialized into a property
- **@JsonPropertyOrder**: Set the order of properties in the serialized JSON output (Not common)
- **@JsonRawValue**: Serialize the string value of the property. WARNING: Can lead to invalid JSON.
- **@JsonValue**: Used to mark a method for serialization output. Will throw exception if more than one method is annotated. WARNING: Can lead to invalid JSON.
- **JsonRootName**: Creates a root element for the object (similar to XML)
- **@JsonSerialize**: Specifies a customer serializer

Deserialization Annotations:
- **@JsonAnySetter**: Converts JSON object into a Java Map object (property names are keys for map)
- **@JsonSetter**: Identifies a Java method to use a setter for the identified JSON property
- **@JsonCreator**: Use identify the object constructor to use (property binding)
- **@JacksonInject**: Allows you to inject default values into properties using deserialization
- **@JsonDeserialize**: Specify customer deserializer

Other:
- **@JsonIgnorePRoperties**: Class-level annotation to ignore one or more properties
- **@JsonIgnore**: Field-level annotation to ignore property
- **@JsonIgnoreType**: Class-level annotation to ignore a class (where the class is a property in other classes).
- **@JsonInclude**: Class level annotation used to control how null values are presented in serialization
- **@JsonAutoDetect**: Jackson will use reflection in serialization. Set which properties/methods are detected. (Rarely used / edge case)

# Property Naming Strategy
Camel case by default

- KEBAB_CASE: kebab-case
- LOWER_CASE: lowercase
- SNAKE_CASE: snake_case
- UPPER_CAMEL_CASE: UpperCamelCase 

Set in application.properties file.  
Can use `@ActiveProfiles("snake")` to configure the class to use a new file.


# Set Property Name 
Any input JSON must have the new id to be read in properly.

```java
public class BeerDto {
    @JsonProperty("beerId") // Sets the JSON name
    @Null
    private UUID id;
}
```


# Custom Serializer (Date Example)
A separate class in the model:
```java
public class LocalDateSerializer extends JsonSerializer<LocalDate> {

    /**
     *
     * @param localDate date to be parsed
     * @param jsonGenerator
     * @param serializerProvider
     * @throws IOException
     */
    @Override
    public void serialize(LocalDate localDate, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeObject(localDate.format(DateTimeFormatter.BASIC_ISO_DATE));
    }
}

```
In the dto:
```java
@JsonSerialize(using=LocalDateSerializer.class)
private LocalDate myLocalDate;
```


# Custom Deserializer (Date Example)
A separate object in the model:
```java
/**
 * Example custom deserializer using Jackson for JSON strings
 */
public class LocalDateDeserializer extends StdDeserializer {

    /**
     * Custom constructor that passes in the class type to the StdDserializer
     */
    protected LocalDateDeserializer() {
        super(LocalDate.class);
    }

    /**
     * Custom deserializer for dates
     * @param jsonParser
     * @param deserializationContext
     * @return
     * @throws IOException
     * @throws JacksonException
     */
    @Override
    public Object deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JacksonException {
        return LocalDate.parse(jsonParser.readValueAs(String.class), DateTimeFormatter.BASIC_ISO_DATE);
    }
}
```

In the dto:
```java
@JsonDeserialize(using=LocalDateDeserializer.class)
private LocalDate myLocalDate;
```


# Jackson JSON Creator for Paged Object
```java
    @JsonCreator(mode = JsonCreator.Mode.PROPERTIES)
    public BeerPagedList(@JsonProperty("content") List<BeerDto> content,
                         @JsonProperty("number") int number,
                         @JsonProperty("size") int size,
                         @JsonProperty("totalElements") Long totalElements,
                         @JsonProperty("pageable") JsonNode pageable,
                         @JsonProperty("last") boolean last,
                         @JsonProperty("totalPages") int totalPages,
                         @JsonProperty("sort") JsonNode sort,
                         @JsonProperty("first") boolean first,
                         @JsonProperty("numberOfElements") int numberOfElements) {

        super(content, PageRequest.of(number, size), totalElements);
    }
```

