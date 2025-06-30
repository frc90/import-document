# Clases e interfazes genericas

## **Indice**

* [1. Patrones comunes en spring boot](#id1)
  * [1.1 - MappedSuperclass](#id1.1)
  * [1.2 - Repositorios Genéricos](#id1.2)
  * [1.3 - Servicios Genéricos (Clase Abstracta)](#id1.3)
  * [1.4 - Mapper base, Dtos necesarios](#id1.4)
  * [1.5 - Controlador]



<div id="id1"/>

## 1. Patrones comunes en spring boot

En Spring boot tenemos un patron repetitivo que siempre esta presente y es la creacion de entidades, repositorios, servicios, controladores, etc....

### Patrones 

<div id="id1.1"/>

### ``1.1 - MappedSuperclass``

Para las entidades y la repeticion de propiedades existe un patrón que te permite reutilizar codigo y se conoce como ``MappedSuperclass`` en **JPA**.

``MappedSuperclass`` para una **BaseEntity**
Una ``MappedSuperclass`` es una clase cuyas propiedades persistentes se heredan a sus subclases de entidad. La clase en sí no es una entidad y no puede ser consultada ni mapeada a una tabla en la base de datos. Solo sus atributos son persistentes cuando se heredan.

```java
package com.example.yourapp.model;

import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.MappedSuperclass;
import jakarta.persistence.Column;
import jakarta.persistence.PrePersist;
import jakarta.persistence.PreUpdate;
import java.time.LocalDateTime;
import java.util.Objects; // Importa Objects para el método equals y hashCode

@MappedSuperclass
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;

    // Métodos para manejar automáticamente created_at y updated_at
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        BaseEntity that = (BaseEntity) o;
        return Objects.equals(id, that.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }

    // Getters y Setters
}
```

## Ventajas de este patrón:
### Reutilización de Código: 
* Evitas repetir los campos id, createdAt, updatedAt y la lógica de manejo de fechas en cada entidad.

### Mantenibilidad: 
* Si necesitas cambiar cómo se genera el ID o cómo se manejan las fechas, solo tienes que modificar la BaseEntity.

### Consistencia: 
* Todas tus entidades tendrán los mismos campos de auditoría, lo que facilita el desarrollo y la depuración.

### Limpieza del Código: 
* Las entidades específicas se centran únicamente en sus propios atributos, mejorando la legibilidad.

* Principios SOLID (DRY - Don't Repeat Yourself): Este patrón es un excelente ejemplo de cómo aplicar el principio DRY



<div id="id1.2">

### ``1.2 - Repositorios Genéricos``

**Spring Data JPA** ya nos ofrece una base fantástica para crear repositorios genéricos. El truco es usar la interfaz **JpaRepository**.

**JpaRepository** como base para tu repositorio genérico
No necesitas crear una interfaz o clase abstracta para un repositorio genérico desde cero. ``JpaRepository<T, ID>`` ya es genérico en sí mismo, donde ``T`` es la entidad y ``ID`` es el tipo de su clave primaria.


```java
// Implementacion 
// @NoRepositoryBean es crucial aquí. Le dice a Spring que no intente crear un bean de esta interfaz directamente.
@NoRepositoryBean
public interface GenericRepository<T extends BaseEntity> extends JpaRepository<T, Long> {
    // Puedes añadir métodos comunes a todos tus repositorios aquí si los necesitas.
    // Por ejemplo, un método para buscar por nombre común si muchas entidades lo tienen.
    // List<T> findByNameContainingIgnoreCase(String name);
}

// Uso
@Repository
public interface ProfesorRepository extends GenericRepository<Profesor> {
    // List<Profesor> findByNombreContainingIgnoreCase(String nombre);
}
```

<div id="id1.3"/>

### ``1.3 - Servicios Genéricos (Clase Abstracta)``
Crear un **servicio** genérico es un poco más manual que con los repositorios, pero es igual de efectivo para el patrón **CRUD**. Aquí usaremos una clase abstracta que encapsule la lógica común.

```java
// Clase abstracta que contendrá la lógica CRUD genérica
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.NoSuchElementException;
import java.util.Optional;
import java.util.stream.Collectors;

public abstract class AbstractService<
        TEntity extends AbstractEntity,
        GRepository extends GenericRepository<TEntity>,
        TResponse,
        TCreateRequest,
        TUpdateRequest
        > {

    public final GRepository repository;
    public final IBaseMapper<TEntity, TResponse, TCreateRequest, TUpdateRequest> mapper;

    protected AbstractService(GRepository repository, IBaseMapper<TEntity, TResponse, TCreateRequest, TUpdateRequest> mapper) {
        this.repository = repository;
        this.mapper = mapper;
    }

    @Transactional(readOnly = true)
    public List<TResponse> findAll(){
        return repository.findAll().stream()
                .map(mapper::toDto)
                .collect(Collectors.toList());
    }

    @Transactional(readOnly = true)
    public Optional<TResponse> findById(Long id){
        return repository.findById(id)
                .map(mapper::toDto);
    }

    @Transactional
    public TResponse save(TCreateRequest createRequest){
        TEntity entityToSave = mapper.toEntity(createRequest);
        TEntity savedEntity = repository.save(entityToSave);
        return mapper.toDto(savedEntity);
    }

    @Transactional
    public boolean delete(Long id){
        if (repository.existsById(id)) {
            repository.deleteById(id);
            return true;
        }
        return false;
    }

    @Transactional(readOnly = true)
    public Page<TResponse> findAllPageable(Pageable pageable){
        return repository.findAll(pageable)
                .map(mapper::toDto);
    }


    @Transactional
    public TResponse update(Long id, TUpdateRequest updateRequest){
        TEntity existingEntity = repository.findById(id)
                .orElseThrow(() -> new NoSuchElementException("Entidad con el ID " + id + " no encontrada para actualizar"));


        mapper.updateEntityFromDto(updateRequest, existingEntity);

        TEntity updatedEntity = repository.save(existingEntity);
        return mapper.toDto(updatedEntity);
    }
}
```

**Como usarlo**:

```java
@Service
public class ItemService extends AbstractService<ItemEntity, ItemRepository, ItemResponseDTO, ItemCreateRequestDTO, ItemUpdateRequestDTO> {
    protected ItemService(ItemRepository repository, IBaseMapper<ItemEntity, ItemResponseDTO, ItemCreateRequestDTO, ItemUpdateRequestDTO> mapper) {
        super(repository, mapper);
    }

    // Aquí puedes añadir métodos de negocio específicos para Item
    // que no sean parte del CRUD genérico, por ejemplo:
    // @Transactional
    // public ItemResponseDTO deactivateItem(Long itemId) {
    //     Item item = repository.findById(itemId)
    //             .orElseThrow(() -> new NoSuchElementException("Item not found: " + itemId));
    //     item.setActive(false);
    //     return mapper.toDto(repository.save(item));
    // }
}
```

<div id="id1.4"/>

### ``1.4 - Mapper base, Dtos necesarios``

Para que todo funcione correctamente hay que crear las clases correspondientes:

* [IBaseMapper](#id1.4.2.1)
* [ItemEntity](#id1.4.2.2)
* [ItemResponseDTO](#id1.4.2.3)
* [ItemCreateRequestDTO](#id1.4.2.4)
* [ItemUpdateRequestDTO](#id1.4.2.5)
* [ItemMapper](#id1.4.2.6)

<div id="id1.4.2.1"/>

``IBaseMapper``
```java
public interface IBaseMapper <TEntity, TResponse, TCreateRequest, TUpdateRequest>{

    TEntity toEntity(TCreateRequest createRequest);

    TResponse toDto(TEntity entity);

    void updateEntityFromDto(TUpdateRequest updateRequest, TEntity entity);
}
```

<div id="id1.4.2.2"/>

Entidad de ejemplo ``ItemEntity``:

```java
@Entity
@Table(name = "items")
@Getter
@Setter
@NoArgsConstructor
public class ItemEntity extends BaseEntity {

    @Column(name = "name", nullable = false, unique = true)
    private String name;

    @Column(name = "description", columnDefinition = "TEXT")
    private String description;

    public ItemEntity(String name, String description) {
        this.name = name;
        this.description = description;
    }
}
```

<div id="id1.4.2.3"/>

``ItemResponseDTO``:

```java
@Getter
@Setter
public class ItemResponseDTO {
    private Long id;
    private String name;
    private String description;
    private boolean active;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

<div id="id1.4.2.4"/>

``ItemCreateRequestDTO``:

```java
import jakarta.validation.constraints.NotBlank;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class ItemCreateRequestDTO {
    @NotBlank(message = "El nombre no puede estar vacío")
    private String name;

    private String description;
}
```

<div id="id1.4.2.5"/>

``ItemUpdateRequestDTO``:

```java
@Getter
@Setter
public class ItemUpdateRequestDTO {
    private String name;
    private String description;
    private Boolean active;
}
```

<div id="id1.4.2.6"/>

#### Finalmente creamos el mapper

`ItemMapper`

```java
import com.example.hello_world.common.mappers.IBaseMapper;
import com.example.hello_world.items.models.ItemEntity;
import org.springframework.stereotype.Component;

@Component
public class ItemMapper implements IBaseMapper<ItemEntity, ItemResponseDTO, ItemCreateRequestDTO, ItemUpdateRequestDTO> {
    @Override
    public ItemEntity toEntity(ItemCreateRequestDTO createRequest) {
        if (createRequest == null) {
            return null;
        }

        ItemEntity item = new ItemEntity();
        item.setName(createRequest.getName());
        item.setDescription(createRequest.getDescription());
        return item;
    }

    @Override
    public ItemResponseDTO toDto(ItemEntity entity) {
        if (entity == null) {
            return null;
        }

        ItemResponseDTO dto = new ItemResponseDTO();
        dto.setId(entity.getId());
        dto.setName(entity.getName());
        dto.setDescription(entity.getDescription());
        dto.setActive(entity.isActive());
        dto.setCreatedAt(entity.getCreatedAt());
        dto.setUpdatedAt(entity.getUpdatedAt());
        return dto;
    }

    @Override
    public void updateEntityFromDto(ItemUpdateRequestDTO updateRequest, ItemEntity entity) {
        if (updateRequest == null || entity == null) {
            return;
        }
        if (updateRequest.getName() != null) {
            entity.setName(updateRequest.getName());
        }
        if (updateRequest.getDescription() != null) {
            entity.setDescription(updateRequest.getDescription());
        }
        if (updateRequest.getActive() != null) {
            entity.setActive(updateRequest.getActive());
        }
    }
}
```

<div id="id1.5"/>

### ``1.5 - Controlador``

Uso de el patron completo en un controlador

```java
@RestController
@RequestMapping("/items")
public class ItemController {

    public final ItemService itemService;

    public ItemController(ItemService itemService) {
        this.itemService = itemService;
    }

    @PostMapping
    public ResponseEntity<ItemResponseDTO> createItem(@Valid @RequestBody ItemCreateRequestDTO createRequest) {
        ItemResponseDTO item = itemService.save(createRequest);
        return new ResponseEntity<>(item, HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<ItemResponseDTO> getItemById(@PathVariable Long id) {
        return itemService.findById(id)
                .map(item -> new ResponseEntity<>(item, HttpStatus.OK))
                .orElseGet(() -> new ResponseEntity<>(HttpStatus.NOT_FOUND));
    }

    @GetMapping
    public ResponseEntity<List<ItemResponseDTO>> getAllItems() {
        List<ItemResponseDTO> items = itemService.findAll();
        return new ResponseEntity<>(items, HttpStatus.OK);
    }

    @PutMapping("/{id}")
    public ResponseEntity<ItemResponseDTO> updateItem(@PathVariable Long id, @Valid @RequestBody ItemUpdateRequestDTO updateRequest) {
        try {
            ItemResponseDTO updatedItem = itemService.update(id, updateRequest);
            return new ResponseEntity<>(updatedItem, HttpStatus.OK);
        } catch (NoSuchElementException e) {
            return new ResponseEntity<>(HttpStatus.NOT_FOUND);
        } catch (Exception e) {
            return new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteItem(@PathVariable Long id) {
        if (itemService.delete(id)) {
            return new ResponseEntity<>(HttpStatus.NO_CONTENT);
        }
        return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
}
```