# Validaciones y manejo de exepciones

**Índice**

1. [Validaciones](#id1)
   - [1.1 Anotaciones de Validaciones](#id1.1)
   - [1.2 Manejador de excepciones global](#id1.2)
   - [1.3 Custom Validation](#id1.3)

# 1. Validaciones

<div id='id1' />

Validaciones es como un control de calidad de los datos. Al igual que un profesor marca las respuestas correctas e incorrectas, la validacion comprueba si la informacion que las personas proporcionan a su programa es correcta y sigue las reglas.

**Spring Boot** ofrece varios mecanismos de validacion, incluidas anotaciones, validaciones personalizadas, gestion de errores y validacion de grupo.

## 1.1 Anotaciones de Validaciones

<div id='id1.1' />

En Spring Boot, la validación se hace más sencilla con anotaciones que marcan campos con reglas de validación específicas. Consideremos un ejemplo de validación de un formulario de registro simple para un usuario:

```java
public class UserRegistrationForm {
    @NotBlank(message = "Please provide a username")
    private String username;
    @Email(message = "Please provide a valid email address")
    private String email;
    @Size(min = 8, message = "Password must be at least 8 characters long")
    private String password;
    // Getters and setters
}
```

1. **@NotNull**: garantiza que un campo no sea nulo.
2. **@NotBlank**: impone la no nulidad y requiere al menos un carácter que no sea un espacio en blanco.
3. **@NotEmpty**: garantiza que las colecciones o matrices no estén vacías.
4. **@Min**(value): comprueba si un campo numérico es mayor o igual que el valor mínimo especificado.
5. **@Max**(value): comprueba si un campo numérico es menor o igual que el valor máximo especificado.
6. **@Size**(min, max): valida si el tamaño de una cadena o colección está dentro de un rango específico.
7. **@Pattern**(regex): verifica si un campo coincide con la expresión regular proporcionada.
8. **@Email**: garantiza que un campo contenga un formato de dirección de correo electrónico válido.
9. **@Digits**(integer, fracture): valida que un campo numérico tenga una cantidad especificada de dígitos enteros y fraccionarios.
10. **@Past** y **@Future**: comprueba que un campo de fecha u hora esté en el pasado y el futuro respectivamente.
11. **@AssertTrue** y **@AssertFalse**: garantizan que un campo booleano sea verdadero y falso respectivamente.
12. **@CreditCardNumber**: validan que un campo contenga un número de tarjeta de crédito válido.
13. **@Valid**: activan la validación de objetos o propiedades anidados.
14. **@Validated**: especifican los grupos de validación que se aplicarán en el nivel de clase o método.

**NOTA:** Cuando le pasas al parametro del metodo **_@Valid_**, Spring boot automaticamente lanza validaciones para los parametros antes que el metodo sea invocado. Se coloca delante del objeto para indicar que debe validarse. Esto significa que los datos que vienen por parametros seran validados con las reglas especificas.

```java
@RestController
@RequestMapping("/user")
public class ApiController {

    @PostMapping("/create")
    public ResponseEntity<String> createUser(@RequestBody @Valid User user, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return ResponseEntity.badRequest().body("Validation failed");
        }
        userService.saveUser(user);
        return ResponseEntity.ok("User created successfully");
    }
}
```

## 1.2 Manejador de excepciones global (Global exception handler)

<div id='id1.2' />

Los errores de validación son inevitables. Spring Boot ofrece una forma de gestionarlos globalmente:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationException(MethodArgumentNotValidException ex) {
        // Create a map of field errors
        // Return appropriate error response
    }
}
```

La anotación **_@ControllerAdvice_** marca una clase como un controlador de excepciones global y manejamos MethodArgumentNotValidException, que se lanza cuando falla la validación.

## 1.3 Custom Validation: Con ejemplo en un Crud para la clase Employee

<div id='id1.3' />

**Exception.java**

```java
public class ResourceConflictException extends RuntimeException {
    public ResourceConflictException(String message) {
        super(message);
    }
}

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}

public class ResponseNotFoundException extends RuntimeException {
    public ResponseNotFoundException(String message) {
        super(message);
    }
}
```

**GlobalExceptionHandler.java**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ApiResponse<Object> handleGeneralException(Exception ex, HttpServletRequest request) {
        return ResponseUtil.error(Arrays.asList(ex.getMessage()), "An unexpected error occurred", 1001, request.getRequestURI());
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ApiResponse<Object> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
        return ResponseUtil.error(Arrays.asList(ex.getMessage()), "Resource not found", 404, request.getRequestURI());
    }

    @ExceptionHandler(ResponseNotFoundException.class)
    public ApiResponse<Object> handleResponseNotFoundException(ResponseNotFoundException ex, HttpServletRequest request) {
        return ResponseUtil.error(Arrays.asList(ex.getMessage()), "Response data not found", 204, request.getRequestURI());
    }
}
```

**Employee.java**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Entity
@Builder
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Size(min = 2, max = 50, message = "You must insert any name.")
    private String username;

    @Size(min = 2, max = 50, message = "You must insert any lastname.")
    private String lastname;

    @Email(message = "Email must be valid.")
    @Column(unique = true)
    private String email;

    @NotNull(message = "You must insert a birthday!")
    private LocalDate birthday;

    @NotNull(message = "You must insert a start date!")
    private LocalDate startDay;

    @NotNull(message = "You must insert is active or not!")
    private Boolean active;
}
```

**EmployeeRespository.java**

```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    Optional<Employee> findByEmail(String email);
}
```

**EmployeeService.java**

```java
public interface EmployeeService {
    List<Employee> getEmployees();
    Employee createEmployee(Employee employee);
    Employee getEmployee(Long id);
    Employee updateEmployee(Long id, Employee employee);
    String deleteEmployee(Long id);
}
```

**EmployeeServiceImpl.java**

```java
@Service
public class EmployeeServiceImpl implements EmployeeService {

    @Autowired
    private EmployeeRepository employeeRepository;

    @Override
    public List<Employee> getEmployees() {
        List<Employee> employees = employeeRepository.findAll();
        if (!employees.isEmpty()) {
            return employees;
        }else {
            throw new ResourceNotFoundException("No employee found");
        }
    }

    @Override
    public Employee createEmployee(Employee employee) {
        if(employeeRepository.findByEmail(employee.getEmail()).isPresent()){
            throw new ResourceConflictException("Email already in use");
        }
        return employeeRepository.save(employee);
    }

    @Override
    public Employee getEmployee(Long id) {
        Optional<Employee> employeeOptional = employeeRepository.findById(id);
        if (employeeOptional.isPresent()) {
            return employeeOptional.get();
        }else{
            throw new ResourceNotFoundException("Employee not found");
        }

    }

    @Override
    @Transactional
    public Employee updateEmployee(Long id, Employee employee) {
        Optional<Employee> employeeOptional = employeeRepository.findById(id);

        if (employeeOptional.isPresent()) {
            Employee existingEmployee = employeeOptional.get();
            existingEmployee.setUsername(employee.getUsername());
            existingEmployee.setLastname(employee.getLastname());
            existingEmployee.setEmail(employee.getEmail());
            existingEmployee.setBirthday(employee.getBirthday());
            existingEmployee.setStartDay(employee.getStartDay());
            existingEmployee.setActive(employee.getActive());
            return employeeRepository.save(existingEmployee);
        }else {
            throw new ResourceNotFoundException("Employee not found with id " + id);
        }
    }

    @Override
    public String deleteEmployee(Long id) {
        if (!employeeRepository.existsById(id)) {
            throw new ResourceNotFoundException("Employee not found with id " + id);
        }
        employeeRepository.deleteById(id);
        return "Employee deleted with id: " + id;
    }
}
```

**EmployeeController.java**

```java
@RestController
@RequestMapping("api/employees")
public class EmployeeController {

    @Autowired
    private EmployeeServiceImpl employeeServiceImpl;

    @GetMapping
    public ResponseEntity<ApiResponse<List<Employee>>> getAllEmployees(HttpServletRequest request) {
        List<Employee> employeesList = employeeServiceImpl.getEmployees();
        return ResponseEntity.ok(ResponseUtil.success(
                        employeesList,
                        "List of employee",
                        request.getRequestURI()
                )
        );
    }

    @PostMapping
    public ResponseEntity<ApiResponse<Employee>> addEmployee(
            @RequestBody @Valid Employee employee,
            BindingResult bindingResult,
            HttpServletRequest request
    ) {
        if (bindingResult.hasErrors()) {
            return ResponseEntity.badRequest().body(ResponseUtil.error(
                    bindingResult.getAllErrors().get(0).getDefaultMessage(),
                    "Error to create employee",
                    400,
                    request.getRequestURI()));
        }
        Employee employeeCreated = employeeServiceImpl.createEmployee(employee);
        return ResponseEntity.ok(ResponseUtil.success(
                employeeCreated,
                "Employee created successfully",
                request.getRequestURI()
        ));
    }

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<Employee>> getEmployee(
            @PathVariable Long id,
            HttpServletRequest request) {

        Employee employee = employeeServiceImpl.getEmployee(id);
        return ResponseEntity.ok(ResponseUtil.success(
                employee,
                "Employee found",
                request.getRequestURI()
        ));
    }

    @PutMapping("/{id}")
    public ResponseEntity<ApiResponse<Employee>> updateEmployee(
            @PathVariable Long id,
            @RequestBody @Valid Employee employee,
            BindingResult bindingResult,
            HttpServletRequest request
    ) {
        if (bindingResult.hasErrors()) {
            return ResponseEntity.badRequest().body(ResponseUtil.error(
                    bindingResult.getAllErrors().get(0).getDefaultMessage(),
                    "Error to update employee",
                    400,
                    request.getRequestURI()
            ));
        }
        Employee updatedEmployee = employeeServiceImpl.updateEmployee(id, employee);
        return ResponseEntity.ok(ResponseUtil.success(
                updatedEmployee,
                "Employee updated successfully",
                request.getRequestURI()
        ));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<ApiResponse<String>> deleteEmployee(@PathVariable Long id, HttpServletRequest request) {
        String value = employeeServiceImpl.deleteEmployee(id);
        return ResponseEntity.ok(ResponseUtil.success(
                value,
                "Employee was deleted successfully",
                request.getRequestURI()
        ));
    }

    @PostMapping("/events")
    public ResponseEntity<ApiResponse<List<EventResponse>>> getEmployeeEvents(
            @RequestBody @Valid EventDateDto eventDate,
            BindingResult bindingResult,
            HttpServletRequest request
    ) {
        if (bindingResult.hasErrors()) {
            return ResponseEntity.badRequest().body(ResponseUtil.error(
                    bindingResult.getAllErrors().get(0).getDefaultMessage(),
                    "Something wrong!!",
                    400,
                    request.getRequestURI()
            ));
        }
        List<EventResponse> eventResponse = employeeServiceImpl.getUpcomingEvents(eventDate);
        return ResponseEntity.ok(ResponseUtil.success(
                eventResponse,
                "Events in a date range",
                request.getRequestURI()
        ));
    }
}
```

**ApiResponse.java**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ApiResponse<T> {
    private boolean success;
    private T data;
    private String message;
    private List<String> errors;
    private int errorCode;
    private long timestamp;
    private String path;
}
```

**ResponseUtil.java**

```java
public class ResponseUtil {

    public static <T> ApiResponse<T> success(T data, String message, String path) {
        ApiResponse<T> response = new ApiResponse<>();
        response.setSuccess(true);
        response.setMessage(message);
        response.setData(data);
        response.setErrors(null);
        response.setErrorCode(0); // No error
        response.setTimestamp(System.currentTimeMillis());
        response.setPath(path);
        return response;
    }

    public static <T> ApiResponse<T> error(List<String> errors, String message, int errorCode, String path) {
        ApiResponse<T> response = new ApiResponse<>();
        response.setSuccess(false);
        response.setMessage(message);
        response.setData(null);
        response.setErrors(errors);
        response.setErrorCode(errorCode);
        response.setTimestamp(System.currentTimeMillis());
        response.setPath(path);
        return response;
    }

    public static <T> ApiResponse<T> error(String error, String message, int errorCode, String path) {
        return error(Arrays.asList(error), message, errorCode, path);
    }
}
```
