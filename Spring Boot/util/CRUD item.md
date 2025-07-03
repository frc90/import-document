Construir un **CRUD completo y autocontenido para la entidad `Book`**, utilizando **DTOs y mappers manuales**, tal como lo harías si estuvieras construyendo un módulo específico de tu aplicación.

-----

## CRUD Específico para `Book` (Sin Nada Genérico)

Vamos a crear un módulo completo para gestionar libros, incluyendo:

  * **Entidad:** `Book` (el modelo de datos persistente).
  * **DTOs:** `BookResponseDTO`, `BookCreateRequestDTO`, `BookUpdateRequestDTO` (para la comunicación con el cliente).
  * **Mapper:** `BookMapper` (clase manual para convertir entre DTOs y la entidad).
  * **Repositorio:** `BookRepository` (interfaz para la interacción con la base de datos).
  * **Servicio:** `BookService` (contiene la lógica de negocio).
  * **Controlador:** `BookController` (expone la API REST).

Asegúrate de que tu `pom.xml` incluya `spring-boot-starter-data-jpa`, `spring-boot-starter-web`, `spring-boot-starter-validation`, y `lombok` (si lo estás usando). Si no, los errores de compilación te lo indicarán.

### 1\. La Entidad `Book`

Esta es la representación de tu tabla en la base de datos. Tendrá las validaciones de Jakarta Bean Validation.

```java
// src/main/java/com/example/demo/book/models/Book.java
package com.example.demo.book.models;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.PrePersist;
import jakarta.persistence.PreUpdate;
import jakarta.persistence.Table;
import jakarta.validation.constraints.DecimalMin;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

import java.time.LocalDateTime;
import java.util.Objects;

@Entity
@Table(name = "books")
@Getter
@Setter
@NoArgsConstructor // Necesario para JPA
public class Book { // Ya no extiende BaseEntity para este ejemplo totalmente independiente

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

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
        active = true; // Por defecto, un libro nuevo está activo
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    @NotBlank(message = "El título no puede estar vacío")
    @Size(max = 255, message = "El título no puede exceder los 255 caracteres")
    @Column(name = "title", nullable = false)
    private String title;

    @NotBlank(message = "El autor no puede estar vacío")
    @Size(max = 255, message = "El autor no puede exceder los 255 caracteres")
    @Column(name = "author", nullable = false)
    private String author;

    @NotNull(message = "El ISBN no puede ser nulo")
    @Size(min = 10, max = 13, message = "El ISBN debe tener entre 10 y 13 caracteres")
    @Column(name = "isbn", unique = true, nullable = false)
    private String isbn;

    @NotNull(message = "El precio no puede ser nulo")
    @DecimalMin(value = "0.0", inclusive = false, message = "El precio debe ser mayor que 0")
    @Column(name = "price", nullable = false)
    private Double price;

    @Min(value = 1, message = "El número de páginas debe ser al menos 1")
    @Column(name = "pages")
    private Integer pages; // Puede ser nulo, por eso no es @NotNull

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Book book = (Book) o;
        return Objects.equals(id, book.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

-----

### 2\. DTOs para `Book`

Estos son los objetos que se usarán para la entrada y salida de datos a través de la API. Las validaciones irán aquí, **no en la entidad**, para desacoplar la capa de presentación de la capa de persistencia.

```java
// src/main/java/com/example/demo/book/dtos/BookResponseDTO.java
package com.example.demo.book.dtos;

import lombok.Getter;
import lombok.Setter;
import java.time.LocalDateTime;

@Getter
@Setter
public class BookResponseDTO {
    private Long id;
    private String title;
    private String author;
    private String isbn;
    private Double price;
    private Integer pages;
    private boolean active;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

```java
// src/main/java/com/example/demo/book/dtos/BookCreateRequestDTO.java
package com.example.demo.book.dtos;

import jakarta.validation.constraints.DecimalMin;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class BookCreateRequestDTO {
    // Validaciones en el DTO de creación
    @NotBlank(message = "El título no puede estar vacío")
    @Size(max = 255, message = "El título no puede exceder los 255 caracteres")
    private String title;

    @NotBlank(message = "El autor no puede estar vacío")
    @Size(max = 255, message = "El autor no puede exceder los 255 caracteres")
    private String author;

    @NotNull(message = "El ISBN no puede ser nulo")
    @Size(min = 10, max = 13, message = "El ISBN debe tener entre 10 y 13 caracteres")
    private String isbn;

    @NotNull(message = "El precio no puede ser nulo")
    @DecimalMin(value = "0.0", inclusive = false, message = "El precio debe ser mayor que 0")
    private Double price;

    @Min(value = 1, message = "El número de páginas debe ser al menos 1")
    private Integer pages; // Opcional
}
```

```java
// src/main/java/com/example/demo/book/dtos/BookUpdateRequestDTO.java
package com.example.demo.book.dtos;

import jakarta.validation.constraints.DecimalMin;
import jakarta.validation.constraints.Min;
import jakarta.validation.constraints.Size;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class BookUpdateRequestDTO {
    // Para actualizaciones parciales, los campos no son @NotBlank ni @NotNull
    // a menos que sean estrictamente obligatorios en cualquier actualización.
    @Size(max = 255, message = "El título no puede exceder los 255 caracteres")
    private String title;

    @Size(max = 255, message = "El autor no puede exceder los 255 caracteres")
    private String author;

    @Size(min = 10, max = 13, message = "El ISBN debe tener entre 10 y 13 caracteres")
    private String isbn;

    @DecimalMin(value = "0.0", inclusive = false, message = "El precio debe ser mayor que 0")
    private Double price;

    @Min(value = 1, message = "El número de páginas debe ser al menos 1")
    private Integer pages;

    private Boolean active; // Permitir cambiar el estado de activo/inactivo
}
```

-----

### 3\. El Mapper Manual para `Book`

Este mapper contendrá la lógica manual para convertir entre DTOs y la entidad.

```java
// src/main/java/com/example/demo/book/mappers/BookMapper.java
package com.example.demo.book.mappers;

import com.example.demo.book.dtos.BookCreateRequestDTO;
import com.example.demo.book.dtos.BookResponseDTO;
import com.example.demo.book.dtos.BookUpdateRequestDTO;
import com.example.demo.book.models.Book;
import org.springframework.stereotype.Component; // Para que Spring lo pueda inyectar

@Component
public class BookMapper {

    // Convierte un BookCreateRequestDTO a una entidad Book
    public Book toEntity(BookCreateRequestDTO createRequestDTO) {
        if (createRequestDTO == null) {
            return null;
        }
        Book book = new Book();
        book.setTitle(createRequestDTO.getTitle());
        book.setAuthor(createRequestDTO.getAuthor());
        book.setIsbn(createRequestDTO.getIsbn());
        book.setPrice(createRequestDTO.getPrice());
        book.setPages(createRequestDTO.getPages());
        // createdAt, updatedAt, active se manejan por @PrePersist en la entidad Book
        return book;
    }

    // Convierte una entidad Book a un BookResponseDTO
    public BookResponseDTO toDto(Book book) {
        if (book == null) {
            return null;
        }
        BookResponseDTO dto = new BookResponseDTO();
        dto.setId(book.getId());
        dto.setTitle(book.getTitle());
        dto.setAuthor(book.getAuthor());
        dto.setIsbn(book.getIsbn());
        dto.setPrice(book.getPrice());
        dto.setPages(book.getPages());
        dto.setActive(book.isActive());
        dto.setCreatedAt(book.getCreatedAt());
        dto.setUpdatedAt(book.getUpdatedAt());
        return dto;
    }

    // Aplica las actualizaciones de un BookUpdateRequestDTO a una entidad Book existente
    public void updateEntityFromDto(BookUpdateRequestDTO updateRequestDTO, Book existingBook) {
        if (updateRequestDTO == null || existingBook == null) {
            return;
        }

        // Solo actualizamos si el campo no es null en el DTO de solicitud
        if (updateRequestDTO.getTitle() != null) {
            existingBook.setTitle(updateRequestDTO.getTitle());
        }
        if (updateRequestDTO.getAuthor() != null) {
            existingBook.setAuthor(updateRequestDTO.getAuthor());
        }
        if (updateRequestDTO.getIsbn() != null) {
            existingBook.setIsbn(updateRequestDTO.getIsbn());
        }
        if (updateRequestDTO.getPrice() != null) {
            existingBook.setPrice(updateRequestDTO.getPrice());
        }
        if (updateRequestDTO.getPages() != null) {
            existingBook.setPages(updateRequestDTO.getPages());
        }
        if (updateRequestDTO.getActive() != null) {
            existingBook.setActive(updateRequestDTO.getActive());
        }
        // Las fechas y el ID no se actualizan desde el DTO.
    }
}
```

-----

### 4\. El Repositorio `BookRepository`

```java
// src/main/java/com/example/demo/book/repositories/BookRepository.java
package com.example.demo.book.repositories;

import com.example.demo.book.models.Book;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface BookRepository extends JpaRepository<Book, Long> {
    Optional<Book> findByIsbn(String isbn); // Ejemplo de método personalizado
}
```

-----

### 5\. El Servicio `BookService`

Este servicio contendrá la lógica de negocio y usará el repositorio y el mapper.

```java
// src/main/java/com/example/demo/book/services/BookService.java
package com.example.demo.book.services;

import com.example.demo.book.dtos.BookCreateRequestDTO;
import com.example.demo.book.dtos.BookResponseDTO;
import com.example.demo.book.dtos.BookUpdateRequestDTO;
import com.example.demo.book.mappers.BookMapper;
import com.example.demo.book.models.Book;
import com.example.demo.book.repositories.BookRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.NoSuchElementException;
import java.util.Optional;
import java.util.stream.Collectors;

@Service
public class BookService {

    private final BookRepository bookRepository;
    private final BookMapper bookMapper;

    public BookService(BookRepository bookRepository, BookMapper bookMapper) {
        this.bookRepository = bookRepository;
        this.bookMapper = bookMapper;
    }

    @Transactional // Marca el método como una transacción de base de datos
    public BookResponseDTO createBook(BookCreateRequestDTO createRequestDTO) {
        // 1. Mapear DTO a entidad
        Book bookToSave = bookMapper.toEntity(createRequestDTO);
        
        // 2. Aquí podrías añadir lógica de negocio adicional antes de guardar,
        //    ej. verificar si el ISBN ya existe, aunque @Column(unique=true) ya lo maneja
        Optional<Book> existingBook = bookRepository.findByIsbn(bookToSave.getIsbn());
        if (existingBook.isPresent()) {
            throw new IllegalArgumentException("Ya existe un libro con el ISBN: " + bookToSave.getIsbn());
        }

        // 3. Guardar la entidad en la base de datos
        Book savedBook = bookRepository.save(bookToSave);

        // 4. Mapear la entidad guardada a DTO de respuesta y devolver
        return bookMapper.toDto(savedBook);
    }

    @Transactional(readOnly = true) // Solo lectura, más eficiente para consultas
    public List<BookResponseDTO> getAllBooks() {
        // 1. Obtener todas las entidades
        List<Book> books = bookRepository.findAll();
        // 2. Mapear cada entidad a DTO de respuesta
        return books.stream()
                .map(bookMapper::toDto)
                .collect(Collectors.toList());
    }

    @Transactional(readOnly = true)
    public Optional<BookResponseDTO> getBookById(Long id) {
        // 1. Buscar la entidad por ID
        // 2. Si se encuentra, mapear a DTO de respuesta; si no, devolver Optional.empty()
        return bookRepository.findById(id)
                .map(bookMapper::toDto);
    }

    @Transactional
    public BookResponseDTO updateBook(Long id, BookUpdateRequestDTO updateRequestDTO) {
        // 1. Buscar la entidad existente por ID
        Book existingBook = bookRepository.findById(id)
                .orElseThrow(() -> new NoSuchElementException("Libro con ID " + id + " no encontrado para actualizar."));

        // 2. Aplicar las actualizaciones desde el DTO a la entidad existente usando el mapper
        //    El mapper maneja la lógica de no sobrescribir con nulls.
        bookMapper.updateEntityFromDto(updateRequestDTO, existingBook);

        // 3. (Opcional) Lógica de negocio adicional después de aplicar actualizaciones pero antes de guardar
        //    Ej. si el ISBN cambia, verificar unicidad del nuevo ISBN
        if (updateRequestDTO.getIsbn() != null && !updateRequestDTO.getIsbn().equals(existingBook.getIsbn())) {
             Optional<Book> bookWithNewIsbn = bookRepository.findByIsbn(updateRequestDTO.getIsbn());
             if (bookWithNewIsbn.isPresent() && !bookWithNewIsbn.get().getId().equals(existingBook.getId())) {
                 throw new IllegalArgumentException("El nuevo ISBN ya está en uso por otro libro.");
             }
        }

        // 4. Guardar la entidad actualizada
        Book updatedBook = bookRepository.save(existingBook);

        // 5. Mapear la entidad actualizada a DTO de respuesta y devolver
        return bookMapper.toDto(updatedBook);
    }

    @Transactional
    public void deleteBook(Long id) {
        // 1. Verificar si el libro existe antes de intentar borrarlo
        if (!bookRepository.existsById(id)) {
            throw new NoSuchElementException("Libro con ID " + id + " no encontrado para eliminar.");
        }
        // 2. Eliminar el libro
        bookRepository.deleteById(id);
    }
}
```

-----

### 6\. El Controlador `BookController`

El controlador manejará las solicitudes HTTP, invocará al servicio y manejará las respuestas y excepciones.

```java
// src/main/java/com/example/demo/book/controllers/BookController.java
package com.example.demo.book.controllers;

import com.example.demo.book.dtos.BookCreateRequestDTO;
import com.example.demo.book.dtos.BookResponseDTO;
import com.example.demo.book.dtos.BookUpdateRequestDTO;
import com.example.demo.book.services.BookService;
import jakarta.validation.Valid; // Para activar las validaciones en los DTOs
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;

import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.NoSuchElementException;

@RestController
@RequestMapping("/api/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @PostMapping // Maneja POST para crear nuevos libros
    public ResponseEntity<BookResponseDTO> createBook(@Valid @RequestBody BookCreateRequestDTO createRequestDTO) {
        BookResponseDTO savedBook = bookService.createBook(createRequestDTO);
        return new ResponseEntity<>(savedBook, HttpStatus.CREATED);
    }

    @GetMapping // Maneja GET para obtener todos los libros
    public ResponseEntity<List<BookResponseDTO>> getAllBooks() {
        List<BookResponseDTO> books = bookService.getAllBooks();
        return new ResponseEntity<>(books, HttpStatus.OK);
    }

    @GetMapping("/{id}") // Maneja GET para obtener un libro por ID
    public ResponseEntity<BookResponseDTO> getBookById(@PathVariable Long id) {
        return bookService.getBookById(id)
                .map(book -> new ResponseEntity<>(book, HttpStatus.OK))
                .orElseGet(() -> new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }

    @PutMapping("/{id}") // Maneja PUT para actualizar un libro existente
    public ResponseEntity<BookResponseDTO> updateBook(@PathVariable Long id, @Valid @RequestBody BookUpdateRequestDTO updateRequestDTO) {
        try {
            BookResponseDTO updatedBook = bookService.updateBook(id, updateRequestDTO);
            return new ResponseEntity<>(updatedBook, HttpStatus.OK);
        } catch (NoSuchElementException e) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        } catch (IllegalArgumentException e) { // Captura excepciones de validación de negocio
            // Puedes devolver un 400 Bad Request con un mensaje de error específico
            return new ResponseEntity<>(HttpStatus.BAD_REQUEST);
        } catch (Exception e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @DeleteMapping("/{id}") // Maneja DELETE para eliminar un libro
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        try {
            bookService.deleteBook(id);
            return new ResponseEntity<>(HttpStatus.NO_CONTENT);
        } catch (NoSuchElementException e) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        }
    }

    // --- Manejador Global de Excepciones para Validaciones (Bean Validation) ---
    // Este método captura las excepciones MethodArgumentNotValidException
    // que Spring lanza cuando un @Valid @RequestBody falla.
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Map<String, String> handleValidationExceptions(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String fieldName = ((org.springframework.validation.FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return errors;
    }
}
```

-----

### 7\. Configuración de Spring Boot y `application.properties`

La clase principal de tu aplicación (la que tiene `@SpringBootApplication`) no necesita cambios para este ejemplo, ya que las inyecciones de dependencia de Spring funcionan automáticamente.

Asegúrate de que tu `application.properties` (o `application.yml`) esté configurado para una base de datos, por ejemplo, H2 en memoria:

```properties
# src/main/resources/application.properties
spring.h2.console.enabled=true
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update # o create para que cree las tablas al inicio
spring.jpa.show-sql=true # Muestra las sentencias SQL en consola
```

-----

### Cómo Probar este CRUD Específico

1.  **Ejecuta tu aplicación Spring Boot.**

2.  **Usa Postman, Insomnia o `curl`** para interactuar con la API:

      * **Crear Libro (POST):**
        `POST http://localhost:8080/api/books`

        ```json
        {
            "title": "La sombra del viento",
            "author": "Carlos Ruiz Zafón",
            "isbn": "9788408043640",
            "price": 15.50,
            "pages": 560
        }
        ```

          * **Validación de `BookCreateRequestDTO` en acción:**
              * Si `title` es vacío o nulo: `"El título no puede estar vacío"`.
              * Si `price` es `0` o negativo: `"El precio debe ser mayor que 0"`.
              * Si `isbn` es duplicado (después del primer libro): `IllegalArgumentException` desde el servicio (devuelve 400 BAD REQUEST).

      * **Obtener Todos los Libros (GET):**
        `GET http://localhost:8080/api/books`

      * **Obtener Libro por ID (GET):**
        `GET http://localhost:8080/api/books/1` (asumiendo que hay un libro con ID 1)

      * **Actualizar Libro (PUT):**
        `PUT http://localhost:8080/api/books/1`

        ```json
        {
            "title": "La sombra del viento (Nueva Edición)",
            "author": "Carlos Ruiz Zafón",
            "isbn": "9788408043640",
            "price": 18.00,
            "pages": 600,
            "active": false
        }
        ```

          * **Validación de `BookUpdateRequestDTO` en acción:**
              * Si `price` se envía como `0`: `"El precio debe ser mayor que 0"`.
              * Si `isbn` se cambia a un valor que ya existe en otro libro: `IllegalArgumentException` desde el servicio.
          * **Manejo de `null`s en `BookMapper`:** Si envías un JSON como `{"title": "Nuevo Título"}` y omites `author`, `isbn`, `price`, etc., el mapper **no sobrescribirá** los campos existentes de `author`, `isbn`, `price` con `null`. Solo el `title` se actualizará.

      * **Eliminar Libro (DELETE):**
        `DELETE http://localhost:8080/api/books/1`

-----