# 2400033034_FSAD_Skill_exp-9

## Skill 9: Global Exception Handling using @ControllerAdvice - FSAD Lab Experiment

### Overview
Demonstrates global exception handling in Spring Boot using @ControllerAdvice for centralized error management.

### Custom Exception Classes

```java
// Custom exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
    
    public ResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}

public class BadRequestException extends RuntimeException {
    public BadRequestException(String message) {
        super(message);
    }
}
```

### Error Response DTO

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private long timestamp;
    private String path;
    private Map<String, String> errors;
}
```

### Global Exception Handler

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(
            ResourceNotFoundException ex, 
            HttpServletRequest request) {
        ErrorResponse error = new ErrorResponse();
        error.setStatus(HttpStatus.NOT_FOUND.value());
        error.setMessage(ex.getMessage());
        error.setTimestamp(System.currentTimeMillis());
        error.setPath(request.getRequestURI());
        
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(error);
    }
    
    @ExceptionHandler(BadRequestException.class)
    public ResponseEntity<ErrorResponse> handleBadRequestException(
            BadRequestException ex,
            HttpServletRequest request) {
        ErrorResponse error = new ErrorResponse();
        error.setStatus(HttpStatus.BAD_REQUEST.value());
        error.setMessage(ex.getMessage());
        error.setTimestamp(System.currentTimeMillis());
        error.setPath(request.getRequestURI());
        
        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(
            MethodArgumentNotValidException ex,
            HttpServletRequest request) {
        Map<String, String> errors = new HashMap<>();
        
        ex.getBindingResult().getFieldErrors()
            .forEach(error -> errors.put(error.getField(), error.getDefaultMessage()));
        
        ErrorResponse error = new ErrorResponse();
        error.setStatus(HttpStatus.BAD_REQUEST.value());
        error.setMessage("Validation failed");
        error.setTimestamp(System.currentTimeMillis());
        error.setPath(request.getRequestURI());
        error.setErrors(errors);
        
        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobalException(
            Exception ex,
            HttpServletRequest request) {
        ErrorResponse error = new ErrorResponse();
        error.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        error.setMessage("An unexpected error occurred");
        error.setTimestamp(System.currentTimeMillis());
        error.setPath(request.getRequestURI());
        
        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(error);
    }
}
```

### REST Controller using Custom Exceptions

```java
@RestController
@RequestMapping("/api/employees")
public class EmployeeController {
    
    @Autowired
    private EmployeeRepository employeeRepository;
    
    @GetMapping("/{id}")
    public ResponseEntity<Employee> getEmployeeById(@PathVariable Long id) {
        Employee employee = employeeRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException(
                "Employee not found with ID: " + id));
        return ResponseEntity.ok(employee);
    }
    
    @PostMapping
    public ResponseEntity<Employee> createEmployee(
            @Valid @RequestBody Employee employee) {
        if (employee.getSalary() < 0) {
            throw new BadRequestException("Salary cannot be negative");
        }
        Employee saved = employeeRepository.save(employee);
        return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(saved);
    }
}
```

### Exception Handler Priority

1. Specific exceptions are handled before generic ones
2. @ExceptionHandler on controller method takes precedence
3. @ControllerAdvice handles controller-wide exceptions
4. Most specific exception type should be listed first

### Key Annotations

| Annotation | Purpose |
|-----------|----------|
| @ControllerAdvice | Global exception handling |
| @ExceptionHandler | Handle specific exceptions |
| @ResponseStatus | Set HTTP status code |
| @ResponseBody | Convert response to JSON |

### Benefits of Global Exception Handling

- **Centralized Error Management**: Single location for all error handling
- **Consistent Response Format**: All errors follow same structure
- **Reduced Code Duplication**: No need for try-catch in every method
- **Better Maintenance**: Easy to update error handling logic
- **Improved User Experience**: Meaningful error messages

### Example Responses

```json
{
  "status": 404,
  "message": "Employee not found with ID: 5",
  "timestamp": 1641234567890,
  "path": "/api/employees/5",
  "errors": null
}
```

```json
{
  "status": 400,
  "message": "Validation failed",
  "timestamp": 1641234567890,
  "path": "/api/employees",
  "errors": {
    "name": "Name is required",
    "email": "Email format is invalid"
  }
}
```

### Testing Exception Handling

```bash
# Test 404 error
curl http://localhost:8080/api/employees/999

# Test 400 error
curl -X POST http://localhost:8080/api/employees \
  -H "Content-Type: application/json" \
  -d '{"salary": -1000}'
```

### Best Practices

1. **Use Meaningful Messages**: Provide clear error descriptions
2. **Log Exceptions**: Log details for debugging
3. **Avoid Exposing Sensitive Data**: Don't reveal system details
4. **Use Correct HTTP Status Codes**: 404 for not found, 400 for bad request, etc.
5. **Consistent Response Structure**: All errors follow same format
6. **Document Exception Handling**: Clearly document what exceptions can be thrown

### Assessment Criteria

- [ ] Custom exception classes defined
- [ ] ErrorResponse DTO created
- [ ] @ControllerAdvice properly configured
- [ ] Multiple @ExceptionHandler methods
- [ ] Custom exceptions thrown in controllers
- [ ] Validation exception handling
- [ ] Generic exception handler as fallback
- [ ] Error responses tested

### References

- [Spring Boot Exception Handling](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-exceptionhandlers)
- [@ControllerAdvice Documentation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html)
- [Exception Handling Best Practices](https://www.baeldung.com/exception-handling-for-rest-with-spring)

### Author

**Student ID:** 2400033034
**Experiment:** 9 - Global Exception Handling
**Last Updated:** March 2026
