### Solución Recomendada: Usar DTOs y Manejo Global de Excepciones

Para abordar estos problemas y lanzar mensajes claros cuando los campos no se envíen (o sean inválidos), vamos a reestructurar tu código usando **DTOs dedicados para la creación y actualización**, y un **manejo global de excepciones para las validaciones**.

Este enfoque es la mejor práctica en aplicaciones Spring Boot modernas.

-----

### 1\. Entidad `AuthorEntity` (¡Sin Validaciones Aquí\!)

Quitamos todas las anotaciones de validación de la entidad. Las validaciones son una preocupación de la capa de entrada/salida (DTOs), no del modelo de datos persistente.

```java
// src/main/java/com/example/hello_world/biblioteca/models/author/AuthorEntity.java
package com.example.hello_world.biblioteca.models.author;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import java.time.LocalDateTime;
import java.util.Objects; // Añadido para equals/hashCode

@Entity
@Table(name = "autors") // Corregido a "authors" si es un error tipográfico, asumiendo "autors" por ahora
@Getter
@Setter
@NoArgsConstructor
public class AuthorEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name", nullable = false)
    private String name;

    @Column(name = "last_name", nullable = false)
    private String lastName;

    @Column(name = "age", nullable = false)
    private Integer age;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;

    @Column(name = "active", nullable = false)
    private boolean active;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
        active = true;
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // Añadidos equals y hashCode para consistencia
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        AuthorEntity that = (AuthorEntity) o;
        return Objects.equals(id, that.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

-----

### 2\. DTOs para Autor (`AuthorResponseDTO`, `AuthorCreateRequestDTO`, `AuthorUpdateRequestDTO`)

Aquí es donde definirás las validaciones y la estructura de los datos que se envían y reciben.

```java
// src/main/java/com/example/hello_world/biblioteca/dtos/author/AuthorResponseDTO.java
package com.example.hello_world.biblioteca.dtos.author;

import lombok.Getter;
import lombok.Setter;
import java.time.LocalDateTime;

@Getter
@Setter
public class AuthorResponseDTO {
    private Long id;
    private String name;
    private String lastName;
    private Integer age;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private boolean active;
}
```

```java
// src/main/java/com/example/hello_world/biblioteca/dtos/author/AuthorCreateRequestDTO.java
package com.example.hello_world.biblioteca.dtos.author;

import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class AuthorCreateRequestDTO {
    @NotBlank(message = "EL nombre no puede estar vacio")
    @Size(max = 50, message = "El nombre no puede exceder de los 50 caracteres")
    private String name;

    @NotBlank(message = "EL apellido no puede estar vacio")
    @Size(max = 50, message = "El apellido no puede exceder de los 50 caracteres")
    private String lastName;

    @NotNull(message = "La edad no puede ser nula")
    @Min(value=20, message = "La edad tiene que ser mayor de 20")
    private Integer age;
}
```

```java
// src/main/java/com/example/hello_world/biblioteca/dtos/author/AuthorUpdateRequestDTO.java
package com.example.hello_world.biblioteca.dtos.author;

import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.Size;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class AuthorUpdateRequestDTO {
    // Para updates parciales, los campos no son @NotBlank ni @NotNull
    // a menos que sean estrictamente obligatorios en cualquier actualización.
    // Usamos tipos wrapper (String, Integer) para que puedan ser null si no se envían.
    @Size(max = 50, message = "El nombre no puede exceder de los 50 caracteres")
    private String name;

    @Size(max = 50, message = "El apellido no puede exceder de los 50 caracteres")
    private String lastName;

    @Min(value=20, message = "La edad tiene que ser mayor de 20")
    private Integer age; // Puede ser nulo en el DTO si no se envía

    private Boolean active; // Para permitir activar/desactivar
}
```

-----

### 3\. Mapper Manual para `Author`

Creamos un mapper para convertir entre DTOs y la entidad. Este será un `@Component` para que Spring pueda inyectarlo.

```java
// src/main/java/com/example/hello_world/biblioteca/mappers/author/AuthorMapper.java
package com.example.hello_world.biblioteca.mappers.author;

import com.example.hello_world.biblioteca.dtos.author.AuthorCreateRequestDTO;
import com.example.hello_world.biblioteca.dtos.author.AuthorResponseDTO;
import com.example.hello_world.biblioteca.dtos.author.AuthorUpdateRequestDTO;
import com.example.hello_world.biblioteca.models.author.AuthorEntity;
import org.springframework.stereotype.Component;

@Component
public class AuthorMapper {

    public AuthorEntity toEntity(AuthorCreateRequestDTO createRequestDTO) {
        if (createRequestDTO == null) {
            return null;
        }
        AuthorEntity author = new AuthorEntity();
        author.setName(createRequestDTO.getName());
        author.setLastName(createRequestDTO.getLastName());
        author.setAge(createRequestDTO.getAge());
        // createdAt, updatedAt, active se manejan en AuthorEntity con @PrePersist
        return author;
    }

    public AuthorResponseDTO toDto(AuthorEntity authorEntity) {
        if (authorEntity == null) {
            return null;
        }
        AuthorResponseDTO dto = new AuthorResponseDTO();
        dto.setId(authorEntity.getId());
        dto.setName(authorEntity.getName());
        dto.setLastName(authorEntity.getLastName());
        dto.setAge(authorEntity.getAge());
        dto.setCreatedAt(authorEntity.getCreatedAt());
        dto.setUpdatedAt(authorEntity.getUpdatedAt());
        dto.setActive(authorEntity.isActive());
        return dto;
    }

    // Método clave para la actualización parcial
    public void updateEntityFromDto(AuthorUpdateRequestDTO updateRequestDTO, AuthorEntity existingAuthor) {
        if (updateRequestDTO == null || existingAuthor == null) {
            return;
        }

        // Solo actualiza si el valor en el DTO de actualización no es null
        if (updateRequestDTO.getName() != null) {
            existingAuthor.setName(updateRequestDTO.getName());
        }
        if (updateRequestDTO.getLastName() != null) {
            existingAuthor.setLastName(updateRequestDTO.getLastName());
        }
        // Para Integer (wrapper type), también comprueba null. Si el cliente envía 0, se actualizará a 0.
        if (updateRequestDTO.getAge() != null) {
            existingAuthor.setAge(updateRequestDTO.getAge());
        }
        // Para Boolean (wrapper type), también comprueba null. Si el cliente envía false, se actualizará a false.
        if (updateRequestDTO.getActive() != null) {
            existingAuthor.setActive(updateRequestDTO.getActive());
        }
        // id, createdAt, updatedAt no se actualizan desde el DTO.
    }
}
```

-----

### 4\. Repositorio `AuthorRepository`

Se mantiene simple, extendiendo `JpaRepository`.

```java
// src/main/java/com/example/hello_world/biblioteca/repositories/author/AuthorRepository.java
package com.example.hello_world.biblioteca.repositories.author;

import com.example.hello_world.biblioteca.models.author.AuthorEntity;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface AuthorRepository extends JpaRepository<AuthorEntity, Long> {
}
```

-----

### 5\. Servicio `AuthorService`

El servicio ahora trabajará con los DTOs y el mapper.

```java
// src/main/java/com/example/hello_world/biblioteca/services/author/AuthorService.java
package com.example.hello_world.biblioteca.services.author;

import com.example.hello_world.biblioteca.dtos.author.AuthorCreateRequestDTO;
import com.example.hello_world.biblioteca.dtos.author.AuthorResponseDTO;
import com.example.hello_world.biblioteca.dtos.author.AuthorUpdateRequestDTO;
import com.example.hello_world.biblioteca.mappers.author.AuthorMapper;
import com.example.hello_world.biblioteca.models.author.AuthorEntity;
import com.example.hello_world.biblioteca.repositories.author.AuthorRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.NoSuchElementException;
import java.util.Optional;
import java.util.stream.Collectors;

@Service
public class AuthorService {
    private final AuthorRepository authorRepository;
    private final AuthorMapper authorMapper;

    public AuthorService(AuthorRepository authorRepository, AuthorMapper authorMapper) {
        this.authorRepository = authorRepository;
        this.authorMapper = authorMapper;
    }

    @Transactional(readOnly = true)
    public List<AuthorResponseDTO> getAllAuthors() {
        return authorRepository.findAll().stream()
                .map(authorMapper::toDto)
                .collect(Collectors.toList());
    }

    @Transactional(readOnly = true)
    public Optional<AuthorResponseDTO> getAuthorById(Long id) {
        return authorRepository.findById(id)
                .map(authorMapper::toDto);
    }

    @Transactional
    public AuthorResponseDTO createAuthor(AuthorCreateRequestDTO createRequestDTO) {
        AuthorEntity authorToSave = authorMapper.toEntity(createRequestDTO);
        AuthorEntity savedAuthor = authorRepository.save(authorToSave);
        return authorMapper.toDto(savedAuthor);
    }

    @Transactional
    public AuthorResponseDTO updateAuthor(Long id, AuthorUpdateRequestDTO updateRequestDTO) {
        AuthorEntity existingAuthor = authorRepository.findById(id)
                .orElseThrow(() -> new NoSuchElementException("Autor con ID " + id + " no encontrado para actualizar."));

        // Utiliza el mapper para aplicar las actualizaciones desde el DTO
        authorMapper.updateEntityFromDto(updateRequestDTO, existingAuthor);

        AuthorEntity updatedAuthor = authorRepository.save(existingAuthor);
        return authorMapper.toDto(updatedAuthor);
    }

    @Transactional
    public void deleteAuthor(Long id) {
        if (!authorRepository.existsById(id)) {
            throw new NoSuchElementException("Autor con ID " + id + " no encontrado para eliminar.");
        }
        authorRepository.deleteById(id);
    }
}
```

-----

### 6\. Controlador `AuthorController`

El controlador manejará las validaciones de los DTOs y el manejo de errores.

```java
// src/main/java/com/example/hello_world/biblioteca/controllers/author/AuthorController.java
package com.example.hello_world.biblioteca.controllers.author;

import com.example.hello_world.biblioteca.dtos.author.AuthorCreateRequestDTO;
import com.example.hello_world.biblioteca.dtos.author.AuthorResponseDTO;
import com.example.hello_world.biblioteca.dtos.author.AuthorUpdateRequestDTO;
import com.example.hello_world.biblioteca.services.author.AuthorService;
import jakarta.validation.Valid; // Importante para activar las validaciones en los DTOs
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError; // Importa FieldError para un manejo más preciso
import org.springframework.web.bind.MethodArgumentNotValidException; // Excepción de validación
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.NoSuchElementException;

@RestController
@RequestMapping("/authors")
public class AuthorController {
    private final AuthorService authorService;

    public AuthorController(AuthorService authorService) {
        this.authorService = authorService;
    }

    @GetMapping
    public ResponseEntity<List<AuthorResponseDTO>> authorsList() {
        List<AuthorResponseDTO> authors = authorService.getAllAuthors();
        return new ResponseEntity<>(authors, HttpStatus.OK);
    }

    @PostMapping
    public ResponseEntity<AuthorResponseDTO> createAuthor(@Valid @RequestBody AuthorCreateRequestDTO createRequestDTO) {
        // Si hay errores de validación, MethodArgumentNotValidException será lanzada
        // y capturada por el manejador global de excepciones (@ExceptionHandler)
        AuthorResponseDTO createdAuthor = authorService.createAuthor(createRequestDTO);
        return new ResponseEntity<>(createdAuthor, HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<AuthorResponseDTO> getAuthor(@PathVariable Long id) {
        return authorService.getAuthorById(id)
                .map(author -> new ResponseEntity<>(author, HttpStatus.OK))
                .orElseGet(() -> new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }

    @PutMapping("/{id}")
    public ResponseEntity<AuthorResponseDTO> updateAuthor(@PathVariable Long id, @Valid @RequestBody AuthorUpdateRequestDTO updateRequestDTO) {
        try {
            AuthorResponseDTO updatedAuthor = authorService.updateAuthor(id, updateRequestDTO);
            return new ResponseEntity<>(updatedAuthor, HttpStatus.OK);
        } catch (NoSuchElementException e) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
        // Otros tipos de excepciones (ej. por ISBN duplicado en Book, si Author lo tuviera) se pueden añadir aquí
        // o con un @ControllerAdvice.
        // Las excepciones de validación son capturadas por el @ExceptionHandler de abajo.
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteAuthor(@PathVariable Long id) {
        try {
            authorService.deleteAuthor(id);
            return new ResponseEntity<>(HttpStatus.NO_CONTENT); // 204 No Content para borrado exitoso
        } catch (NoSuchElementException e) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
    }

    ---
    ### Manejador Global de Excepciones para Validaciones
    ---
    // Este método captura las excepciones MethodArgumentNotValidException
    // que Spring lanza cuando un @Valid @RequestBody falla.
    @ResponseStatus(HttpStatus.BAD_REQUEST) // Envía un 400 Bad Request
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Map<String, String> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            // FieldError es específico para errores de campo (ej. @NotBlank, @NotNull, @Size)
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return errors; // Devuelve un JSON con los nombres de los campos y sus mensajes de error
    }
}
```

-----

### Cómo Probar los Mensajes de Validación

1.  **Asegúrate de que tu `pom.xml` tiene la dependencia `spring-boot-starter-validation`**.

2.  **Configura tu `application.properties`** con una base de datos (como H2 en memoria).

3.  **Ejecuta tu aplicación Spring Boot.**

4.  **Usa Postman/Insomnia/curl** para enviar solicitudes y ver los mensajes de error:

      * **POST (Crear Autor - Campos faltantes/inválidos):**
        `POST http://localhost:8080/authors`

        ```json
        {
            "name": "",      // Fallará @NotBlank
            "lastName": "Doe",
            "age": 15        // Fallará @Min
        }
        ```

        **Respuesta esperada (400 Bad Request):**

        ```json
        {
            "name": "EL nombre no puede estar vacio",
            "age": "La edad tiene que ser mayor de 20"
        }
        ```

        Aquí verás los mensajes que configuraste en tus DTOs. Si omites un campo como `"name"`, el `Jackson` lo mapeará a `null`, y `NotBlank` lo detectará. Si omites `age`, será `null` y `NotNull` lo detectará.

      * **PUT (Actualizar Autor - Campos faltantes/inválidos):**
        Primero, crea un autor válido para obtener su ID. Luego, intenta actualizarlo.
        `PUT http://localhost:8080/authors/{id}`

        ```json
        {
            "name": "Jane",
            "lastName": "",  // Fallará @Size y @NotBlank si lo incluyes vacío
            "age": 18        // Fallará @Min
        }
        ```

        **Respuesta esperada (400 Bad Request):**

        ```json
        {
            "lastName": "EL apellido no puede estar vacio",
            "age": "La edad tiene que ser mayor de 20"
        }
        ```

        Si en el `PUT` **no incluyes un campo** (por ejemplo, `lastName`), el `AuthorUpdateRequestDTO` lo recibirá como `null`. Como `lastName` en `AuthorUpdateRequestDTO` no tiene `@NotBlank` ni `@NotNull` (porque es una actualización parcial), no se generará un error de validación para ese campo, y tu mapper no lo modificará en la entidad existente. Esto es el comportamiento deseado para actualizaciones parciales.

-----