### Uso de DTOs y Mappers (MapStruct, ModelMapper)

Esta es la **solución preferida y más robusta** en sistemas Spring Boot modernos, especialmente para manejar operaciones de actualización (parciales o completas) y para desacoplar las entidades de la API.

1.  **DTOs (Data Transfer Objects)**: Define clases específicas para la entrada (Request DTOs) y salida (Response DTOs) de datos. Un `ProfesorUpdateRequest` solo contendría los campos que *pueden* ser actualizados.
2.  **Mappers**: Utiliza una librería como **MapStruct** (preferida por su rendimiento y generación de código en tiempo de compilación) o ModelMapper para transformar los DTOs en entidades y viceversa. Los mappers pueden configurarse para ignorar campos `null` o para mapear solo campos específicos.

**Ejemplo con MapStruct (conceptual):**

Primero, el DTO de actualización:

```java
// ProfesorUpdateRequest.java
public class ProfesorUpdateRequest {
    private String nombre;
    private String apellido;
    // No incluyas id, createdAt, updatedAt aquí
}
```

Luego, una interfaz Mapper:

```java
// ProfesorMapper.java
import org.mapstruct.Mapper;
import org.mapstruct.MappingTarget;
import org.mapstruct.NullValuePropertyMappingStrategy;

@Mapper(componentModel = "spring", nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
public interface ProfesorMapper {

    // Método para mapear un DTO de actualización a una entidad existente
    void updateProfesorFromDto(ProfesorUpdateRequest dto, @MappingTarget Profesor entity);
}
```

Configuración de MapStruct en `pom.xml`:

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>${org.mapstruct.version}</version>
</groupId>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>${org.mapstruct.version}</version>
    <scope>provided</scope>
</dependency>
```

Finalmente, cómo se usaría en un servicio **específico** (no genérico para la actualización compleja):

```java
// ProfesorService.java
@Service
public class ProfesorService { // Este no es el AbstractService genérico, es el específico
    private final ProfesorRepository profesorRepository;
    private final ProfesorMapper profesorMapper; // Inyecta el mapper

    public ProfesorService(ProfesorRepository profesorRepository, ProfesorMapper profesorMapper) {
        this.profesorRepository = profesorRepository;
        this.profesorMapper = profesorMapper;
    }

    @Transactional
    public Profesor updateProfesor(Long id, ProfesorUpdateRequest updateRequest) {
        Profesor existingProfesor = profesorRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Profesor con ID " + id + " no encontrado."));

        // Aquí es donde MapStruct hace la magia:
        // Solo copiará las propiedades que estén presentes en updateRequest y no sean null.
        // Las propiedades de updateRequest que no existan en Profesor o que sean null serán ignoradas.
        profesorMapper.updateProfesorFromDto(updateRequest, existingProfesor);

        return profesorRepository.save(existingProfesor);
    }
    // ... otros métodos de ProfesorService ...
}
```

**Cuándo usarlo**: **Siempre que sea posible para APIs RESTful.** Es el estándar de la industria. Proporciona control total sobre qué campos pueden ser actualizados, maneja la conversión de tipos, ignora nulos, y desacopla la representación de la API de tu modelo de dominio. Sin embargo, para que funcione bien con MapStruct o ModelMapper, generalmente se aplica a servicios **específicos** de la entidad, no al `AbstractService` genérico, ya que el mapper necesita conocer los tipos concretos (Profesor, ProfesorUpdateRequest) para generar el código de mapeo.

-----