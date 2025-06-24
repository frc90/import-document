# Clases e interfazes genericas

## **Indice**

* [1. Patrones comunes en spring boot](#id1)
  * [1.1 - MappedSuperclass](#id1.1)
  * [1.2 - Repositorios Genéricos](#id1.2)
  * [1.3 - Servicios Genéricos (Clase Abstracta)](#id1.3)



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
public abstract class AbstractService<T extends BaseEntity, R extends GenericRepository<T>> {

    protected final R repository; // El repositorio específico para la entidad T

    public AbstractService(R repository) {
        this.repository = repository;
    }

    @Transactional(readOnly = true)
    public List<T> findAll() {
        return repository.findAll();
    }

    @Transactional(readOnly = true)
    public Optional<T> findById(Long id) {
        return repository.findById(id);
    }

    @Transactional
    public T save(T entity) {
        return repository.save(entity);
    }

    @Transactional
    public void deleteById(Long id) {
        repository.deleteById(id);
    }

    @Transactional(readOnly = true)
    public Page<T> findAll(Pageable pageable) {
        return repository.findAll(pageable);
    }

    // Puedes añadir más métodos comunes aquí, como actualizar (si necesitas lógica específica de actualización)
    @Transactional
    public T update(Long id, T updatedEntity) {
        return repository.findById(id).map(existingEntity -> {
            // Aquí deberías copiar las propiedades actualizables de updatedEntity a existingEntity.
            // Para un servicio genérico, esto puede ser complicado sin usar reflexión o una interfaz de "mapeador".
            // Para simplicidad en este ejemplo, y porque el 'save' ya hace un 'merge', simplemente retornamos el save.
            // En un caso real, podrías necesitar un método 'update' en la entidad o un Mapper.
            updatedEntity.setId(id); // Asegura que el ID sea el correcto para la actualización
            return repository.save(updatedEntity);
        }).orElseThrow(() -> new RuntimeException("Entidad con ID " + id + " no encontrada para actualizar."));
    }
}
```

**Como usarlo**:

```java
@Service
public class ProfesorService extends AbstractService<Profesor, ProfesorRepository> {

    public ProfesorService(ProfesorRepository repository) {
        super(repository);
    }

    // Aquí puedes añadir métodos específicos de negocio para Profesor
    // Por ejemplo:
    // public List<Profesor> findProfesoresConMuchosEstudiantes() { ... }
}
```