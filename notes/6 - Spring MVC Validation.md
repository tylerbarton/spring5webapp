# Spring MVC Validation
Bean validation used to restrict the data & input provided by the user using standard validation annotations. These are handled in the controller units.

# Java Bean Validation:
- **Important**: Must be brought in an separate dependency 
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```
- Validates input parameters meet set requirements
- Includes dependency injection for bean validation components
- If validation fails, HTTPStatus **400** returned

Popular implementation: Hibernate Validation

Built in Hibernate Constraint Definitions:
- **@Null**: checks value is null
- **@NotNull**: checks value is not null

### Boolean
- **@AssertTrue**: value is true
- **@AssertFalse**: value is false

### Numerical
- **@Min**: value is equal or higher
- **@Max**: value is equal or lower
- **@DecimalMin**: value is higher
- **@DecimalMax**: value is lower
- **@Negative**: value is negative
- **@NegativeOrZero**: value is negative or zero
- **@Positive**: value is negative
- **@PositiveOrZero**: value is positive or zero
- **@Size**: checks if string or collection is between a min and max
- **@Digits**: check for integer digits and fraction digits
- **@Range**: check if number is between given min and max (inclusive)

### Dates
- **@Past**: check if *date* is in past
- **@PastOrPresent**: check if *date* is in past or present
- **@Future**: check if *date* is in future
- **@FutureOrPresent**: check if *date* is in future or present

### Strings
- **@Pattern**: checks against RegEx pattern
- **@NotEmpty**: check if value is not _null_ nor empty (whitespace characters or empty collection)
- **@NonBlank**: Checks string is not null or not whitespace characters
- **@Email**: Checks if string value is an email address
- **Length**: String length between given min and max

### Edge Cases
- **@ScriptAssert**: Class level annotation, checks class against script
- **@CreditCardNumber**: Verifies value is a credit card number
- **@Currency**: Valid currency amount
- **@DurationMax**: Duration is less than given amount
- **@DurationMin**: Duration is greater than given amount
- **@EAN**: Valid EAN barcode (consumer products)
- **@ISBN**: Valid ISBN value (books)
- **@SafeHtml**: Checks for safe HTML
- **@UniqueElements**: Checks a collection for unique elements
- **@Url**: Checks for valid URL (websites)

You can also define custom validators.


# Method Validation:
Validation on methods & controllers

In the controller class add `@Validated` annotation to perform validation on method input parameters.

Not used as much as data POJO validation


# @ControllerAdvice (Global Controller Methods)
If you have blocks of code that are the same (such as validation exception handler).

This annotation provides the code globally to all controllers